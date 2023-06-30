# numa-topology-generic-device

## Prerequisites
- Install RKE2 =>1.25.7
- Nodes must have multiple NUMA nodes per CPU
- setenforce 0
- RKE2 Configuration Options:

```yaml
kubelet-arg:
- feature-gates=MemoryManager=true
- kube-reserved=cpu=400m,memory=2Gi
- system-reserved=cpu=400m,memory=2Gi
- memory-manager-policy=Static
- reserved-memory=0:memory=2Gi
- reserved-memory=1:memory=2Gi
- topology-manager-policy=single-numa-node
```

## Clone this repo

```
git clone https://github.com/rancherfederal/numa-topology-generic-device
cd numa-topology-generic-device
```


## Install the device plugin

```
kubectl apply -k devicepluginA

kubectl get nodes -o yaml | grep -i deviceA -C 4
```


## install NFD

Next, we install the node-feature-discovery addon. 

See more information: https://github.com/kubernetes-sigs/node-feature-discovery/tree/master

```
kubectl apply -k https://github.com/kubernetes-sigs/node-feature-discovery/deployment/overlays/default?ref=v0.13.2
```

Verify it is up and running

```
kubectl -n node-feature-discovery get ds,deploy
```

## install NFD topology updater

Then the nfd-topology-updater. 

```
kubectl apply -k https://github.com/kubernetes-sigs/node-feature-discovery/deployment/overlays/topologyupdater?ref=v0.13.2
```

Verify the NFD Topology updater created the noderesourcetopology CRs

```
kubectl get noderesourcetopology.topology.node.k8s.io -o yaml
```

## Setup the topo-aware-scheduler

On each of the server nodes, copy the rke2.yaml to /etc/kubernetes/scheduler.conf and replace the localhost ip with the primary ip of a master node.

```
cp /etc/rancher/rke2/rke2.yaml /etc/kubernetes/scheduler.conf
vi /etc/kubernetes/scheduler.conf
```

Apply the yamls to deploy the secondary scheduler.

```
cd noderesourcetopology/
kubectl apply -f cluster-role.yaml

kubectl apply -f crd.yaml

kubectl apply -f scheduler-configmap.yaml

kubectl apply -f deploy-scheduler.yaml

```

## Example deployment using the secondary scheduler

```
k apply -f example-test.yaml
```


Verify the resources are showing correctly in the noderesourcetopology

```
k get noderesourcetopology.topology.node.k8s.io -o yaml
```

you should see the output:

```
apiVersion: topology.node.k8s.io/v1alpha2
attributes:
- name: topologyManagerPolicy
value: single-numa-node
- name: topologyManagerScope
value: container
kind: NodeResourceTopology
metadata:
creationTimestamp: "2023-06-28T15:27:28Z"
generation: 23
name: ip-192-167-190-41.us-gov-west-1.compute.internal
resourceVersion: "13307"
uid: 245abaee-9f70-4918-875e-3f890f655d66
topologyPolicies:
- SingleNUMANodeContainerLevel
zones:
- costs:
- name: node-0
value: 10
- name: node-1
value: 21
name: node-0
resources:
- allocatable: "265094635520"
available: "265094635520"
capacity: "267242119168"
name: memory
- allocatable: "10"
available: "7"
capacity: "10"
name: example.com/deviceA
type: Node
- costs:
- name: node-0
value: 21
- name: node-1
value: 10
name: node-1
resources:
- allocatable: "265474244608"
available: "265474244608"
capacity: "267621728256"
name: memory
- allocatable: "10"
available: "10"
capacity: "10"
name: example.com/deviceA
type: Node
```

The deployment requests 3 DeviceA

### example pod deployment requirements
```yaml
resources:
    limits:
    memory: 100Mi
    example.com/deviceA: 3
    requests:
    memory: 100Mi
    example.com/deviceA: 3
```


### Verify noderesourcetopology.topology.node.k8s.io shows the updated resources

```yaml
- allocatable: "10"
available: "7"
capacity: "10"
name: example.com/deviceA
```

