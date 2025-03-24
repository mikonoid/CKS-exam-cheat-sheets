# Quality of Service (QoS) in Kubernetes

## What is Kubernetes QoS?
Quality of Service (QoS) in Kubernetes ensures that Pods get the appropriate amount of CPU and memory based on their resource requests and limits. It helps in **scheduling, resource management, and eviction decisions**.

## Why is QoS Important for CKS?

 - Prevents resource starvation of critical workloads.
 - Helps optimize cluster performance.
 - Ensures high-priority workloads get guaranteed resources.
 -  Reduces risk of eviction due to resource contention.

## Kubernetes QoS Classes
Kubernetes assigns one of the following QoS classes to a Pod based on its resource requests and limits:

### 1. **Guaranteed** (Highest Priority)
- The Pod gets **dedicated** CPU/memory resources.
- Both **requests and limits must be equal and set** for all containers.
- The Pod is **least likely to be evicted**.

#### Example: Guaranteed QoS Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 2. **Burstable** (Medium Priority)
- The Pod has **requests lower than limits**.
- It can consume more resources **when available**.
- It **may get evicted** if the node runs out of resources.

#### Example: Burstable QoS Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 3. **BestEffort** (Lowest Priority)
- **No resource requests or limits are set**.
- The Pod **consumes resources opportunistically**.
- It is **first to be evicted** if the node runs out of resources.

#### Example: BestEffort QoS Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: besteffort-pod
spec:
  containers:
  - name: app
    image: nginx
```

## Checking QoS Class of a Pod
```sh
kubectl get pod <pod-name> -o=jsonpath='{.status.qosClass}'
```

## Eviction Behavior Based on QoS
- **BestEffort** Pods are evicted **first** during memory pressure.
- **Burstable** Pods are evicted **next**, based on priority and resource usage.
- **Guaranteed** Pods are **evicted last**, only if the node cannot reclaim enough resources.

## Best Practices for Managing QoS in Kubernetes
✅ **Use Guaranteed QoS for mission-critical workloads.**
✅ **Set proper requests and limits for all Pods to avoid resource starvation.**
✅ **Monitor node utilization with `kubectl top nodes`.**
✅ **Use `kubectl describe node` to check eviction conditions.**
✅ **Enable resource quotas and limits to enforce resource policies.**

---
**Reference**: [Kubernetes QoS Documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#quality-of-service-classes)
