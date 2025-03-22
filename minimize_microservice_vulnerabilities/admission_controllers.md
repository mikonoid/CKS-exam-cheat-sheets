# Admission Controllers

## What are Admission Controllers?
Admission Controllers are plugins that intercept and modify (or reject) requests to the Kubernetes API before they are persisted to etcd.

## Types of Admission Controllers
1. **Mutating Admission Controllers**: Modify the request before validation.
2. **Validating Admission Controllers**: Only validate the request and reject if necessary.

## Enabling Admission Controllers
Admission controllers are enabled via the `--enable-admission-plugins` flag in the **kube-apiserver**.

Example:
```sh
--enable-admission-plugins=NamespaceLifecycle,PodSecurityPolicy,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
```

## Common Security-Related Admission Controllers
| Admission Controller         | Type       | Purpose |
|-----------------------------|-----------|---------|
| `PodSecurity`               | Validating | Enforces Pod Security Standards (PSS) |
| `NodeRestriction`           | Validating | Restricts kubelet from modifying critical API objects |
| `SecurityContextDeny`       | Validating | Blocks insecure securityContext settings |
| `AlwaysPullImages`          | Mutating   | Forces pulling images from registry to prevent local image usage |
| `MutatingAdmissionWebhook`  | Mutating   | Allows dynamic admission policies via webhooks |
| `ValidatingAdmissionWebhook`| Validating | Allows validation policies via webhooks |

## Webhook-Based Admission Controllers
- **MutatingAdmissionWebhook**
- **ValidatingAdmissionWebhook**

### Webhook Configuration Example
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: example-webhook
webhooks:
  - name: validate.example.com
    rules:
      - apiGroups: ["*"]
        apiVersions: ["*"]
        operations: ["CREATE", "UPDATE"]
        resources: ["pods"]
    clientConfig:
      service:
        name: webhook-service
        namespace: default
        path: "/validate"
      caBundle: Cg==
    admissionReviewVersions: ["v1"]
    sideEffects: None
```

## Best Practices for Admission Controllers
- Use `PodSecurity` for enforcing security policies.
- Enable `NodeRestriction` to prevent kubelet from escalating privileges.
- Implement **webhook-based controllers** for custom policies.
- Ensure **high availability** of webhooks to prevent API failures.
- Set `failurePolicy: Ignore|Fail` wisely to control behavior on webhook failures.

## Debugging Admission Controllers
- List enabled controllers:
  ```sh
  kubectl api-resources --verbs=list | grep admission
  ```
- Check webhook logs:
  ```sh
  kubectl logs -n kube-system -l app=admission-webhook
  ```
- Inspect API server logs:
  ```sh
  journalctl -u kube-apiserver
  ```

---
**Reference**: [Kubernetes Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
