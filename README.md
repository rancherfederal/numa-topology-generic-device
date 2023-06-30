# numa-topology-generic-device

## Prerequisites
- Install RKE2 =>1.25.7
- Nodes must have multiple NUMA nodes per CPU
- RKE2 Configuration Options:

```
kubelet-arg:
- feature-gates=MemoryManager=true
- kube-reserved=cpu=400m,memory=2Gi
- system-reserved=cpu=400m,memory=2Gi
- memory-manager-policy=Static
- reserved-memory=0:memory=2Gi
- reserved-memory=1:memory=2Gi
- topology-manager-policy=single-numa-node
```

