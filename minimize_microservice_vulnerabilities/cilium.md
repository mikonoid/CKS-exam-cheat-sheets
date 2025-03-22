# Cilium

## What is Cilium?
Cilium is an open-source, eBPF-based networking and security solution for Kubernetes. It provides **network observability, security policies, and load balancing** with minimal performance overhead.

## Why Use Cilium for Kubernetes Security?
✅ Uses **eBPF** for high-performance packet processing.
✅ Provides **L3-L7 Network Policies** beyond traditional Kubernetes NetworkPolicies.
✅ Supports **Pod-to-Pod Encryption** via WireGuard.
✅ Enables **DNS-aware policies** to restrict external traffic.
✅ Supports **Hubble** for deep network observability.

## Installing Cilium in Kubernetes
```sh
helm repo add cilium https://helm.cilium.io
helm repo update
helm install cilium cilium/cilium --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set hubble.enabled=true \
  --set security.enabled=true \
  --set encryption.enabled=true
```

## Configuring Cilium Network Policies
Cilium extends Kubernetes **NetworkPolicies** with:
- **L3/L4 policies** (IP-based restrictions)
- **L7 policies** (HTTP, Kafka, DNS filtering)
- **CIDR-based Egress rules**

### Example: Deny All Traffic by Default
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-all
spec:
  endpointSelector: {}
  ingress: []
  egress: []
```

### Example: Allow Pod-to-Pod Communication on HTTP
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-http
spec:
  endpointSelector:
    matchLabels:
      app: frontend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: backend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: GET
          path: "/api"
```

## Enabling Pod-to-Pod Encryption with WireGuard
```sh
helm upgrade cilium cilium/cilium --namespace kube-system --set encryption.enabled=true --set encryption.type=wireguard
```
Verify encryption status:
```sh
kubectl exec -it <pod> -- ip xfrm state
```

## Monitoring with Hubble (Cilium Observability)
### Install Hubble CLI
```sh
curl -L --remote-name https://github.com/cilium/hubble/releases/latest/download/hubble-linux-amd64
chmod +x hubble-linux-amd64
mv hubble-linux-amd64 /usr/local/bin/hubble
```
### Enable Hubble in Cilium
```sh
helm upgrade cilium cilium/cilium --namespace kube-system --set hubble.enabled=true
```
### View Network Flows in Real-Time
```sh
hubble observe --namespace default
```

## Troubleshooting Cilium
### Check Cilium DaemonSet
```sh
kubectl get pods -n kube-system -l k8s-app=cilium
```
### Validate Cilium Status
```sh
cilium status
```
### Debug Policy Drops
```sh
hubble observe --drop
```

## Summary
✅ **Cilium replaces traditional Kubernetes networking with eBPF-based security.**
✅ **Network policies enforce L3-L7 security controls.**
✅ **WireGuard enables Pod-to-Pod encryption.**
✅ **Hubble provides real-time network observability.**
✅ **Cilium improves performance while maintaining security.**

---
**Reference**: [Cilium Official Documentation](https://docs.cilium.io/)
