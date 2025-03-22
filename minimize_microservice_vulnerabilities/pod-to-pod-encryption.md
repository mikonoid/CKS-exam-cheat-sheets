# Pod-to-Pod Encryption

## What is Pod-to-Pod Encryption?
Pod-to-Pod encryption ensures that data transmitted between Kubernetes pods remains secure and protected from eavesdropping or man-in-the-middle attacks.

## Why is Pod-to-Pod Encryption Important?
- Protects sensitive data in transit.
- Prevents network sniffing and unauthorized access.
- Essential for compliance with security standards like GDPR and HIPAA.

## Methods for Implementing Pod-to-Pod Encryption

### 1. **mTLS with Service Mesh (Istio/Linkerd)**
- Enables **mutual TLS (mTLS)** encryption between services.
- Provides automatic certificate management and rotation.
- Enforces authentication and authorization for service communication.

#### Example: Enabling mTLS in Istio
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

#### Verify mTLS Status
```sh
istioctl authn tls-check <pod>.<namespace>
```

### 2. **IPsec with Cilium or Calico**
- Uses **IPsec (ESP)** or **WireGuard** to encrypt pod-to-pod traffic at the network level.
- Works at Layer 3 (IP layer), independent of applications.

#### Enable WireGuard in Calico
```sh
kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"wireguardEnabled":true}}'
```

#### Verify Encryption
```sh
kubectl exec -it <pod> -- tcpdump -i eth0 -n port 443
```

### 3. **TLS for Application-Level Encryption**
- Encrypts HTTP/gRPC traffic using TLS certificates.
- Requires services to support **HTTPS** or **gRPC TLS**.

#### Example: Enforcing HTTPS in a Kubernetes Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
  - hosts:
      - example.com
    secretName: tls-secret
```

### 4. **Enforcing Network Policies with Encryption**
- Define **NetworkPolicies** to restrict non-encrypted traffic.
- Works with CNI plugins like Cilium, Calico, or Weave.

#### Example: Allow Only TLS Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-tls-only
  namespace: default
spec:
  podSelector: {}
  ingress:
    - ports:
        - port: 443
          protocol: TCP
```

## Verifying Pod-to-Pod Encryption
- Check Istio mTLS status:
  ```sh
  istioctl proxy-config listeners <pod>.<namespace>
  ```
- Inspect encrypted traffic with **tcpdump**:
  ```sh
  kubectl exec -it <pod> -- tcpdump -i eth0 -n
  ```
- Ensure WireGuard encryption is enabled:
  ```sh
  calicoctl get felixconfig -o yaml | grep wireguardEnabled
  ```

## Summary
✅ Use **mTLS with Istio/Linkerd** for service-to-service encryption.
✅ Enable **IPsec or WireGuard** for network-layer encryption.
✅ Enforce **TLS at the application level** for HTTP/gRPC traffic.
✅ Restrict non-encrypted traffic using **NetworkPolicies**.
✅ Monitor encrypted traffic with **tcpdump** and network observability tools.

---
**Reference**: [Kubernetes Network Security](https://kubernetes.io/docs/concepts/security/network-policies/)
