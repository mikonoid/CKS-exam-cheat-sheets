# Image Security

## What is Image Security?
Image security in Kubernetes ensures that container images are **trusted, verified, and free from vulnerabilities** before deployment. It helps prevent **supply chain attacks, unauthorized modifications, and runtime risks**.

## Why is Image Security Important for CKS?
✅ Prevents **container image tampering** and unauthorized modifications.
✅ Reduces the risk of **vulnerable dependencies** in production.
✅ Ensures compliance with **security best practices** (e.g., CIS, SLSA, NIST SSDF).
✅ Protects Kubernetes clusters from **malicious images**.

---
## Key Aspects of Image Security

### 1. **Use Trusted and Minimal Base Images**
- Prefer **official, verified images** from trusted sources (e.g., `alpine`, `distroless`).
- Use minimal images to **reduce the attack surface**.
- Avoid `latest` tags – use **immutable versions**.

#### Example: Using a Minimal Base Image
```Dockerfile
FROM gcr.io/distroless/static:nonroot
```

### 2. **Scan Images for Vulnerabilities**
- Use security scanners to detect CVEs in container images.
- Popular tools: **Trivy, Grype, Snyk, Clair**.

#### Example: Scan an Image with Trivy
```sh
trivy image my-registry.io/my-image:latest
```

### 3. **Sign and Verify Images**
- Use **Sigstore Cosign** to sign and verify container images.
- Ensures only **trusted images** are deployed.

#### Example: Signing an Image
```sh
cosign sign --key cosign.key my-registry.io/my-image:latest
```

#### Example: Verifying an Image Signature
```sh
cosign verify my-registry.io/my-image:latest --key cosign.pub
```

### 4. **Secure Docker Image Build Practices**
- Use **multi-stage builds** to keep images clean.
- Avoid **embedding secrets** in Dockerfiles.
- Reduce **privileged access** by using non-root users.

#### Example: Multi-Stage Build
```Dockerfile
FROM golang:1.18 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/myapp /
CMD ["/myapp"]
```

### 5. **Store Images Securely in a Private Registry**
- Use **private container registries** (e.g., Docker Hub, Amazon ECR, Google Artifact Registry, Harbor).
- Enable **access controls and scanning**.
- Avoid exposing registries to the public.

#### Example: Enable Scanning in AWS ECR
```sh
aws ecr put-image-scanning-configuration --repository-name my-repo --image-scanning-configuration scanOnPush=true
```

---
## Managing Secrets in Kubernetes for Private Docker Repositories
### 6. **Secure Access to Private Registries**
- Require **authentication** for pulling images.
- Use **Kubernetes Secrets** to store registry credentials.
- Enforce **role-based access control (RBAC)** for registry access.

#### Example: Creating a Kubernetes Secret for a Private Registry
```sh
kubectl create secret docker-registry my-registry-secret \
  --docker-server=my-registry.io \
  --docker-username=my-user \
  --docker-password=my-password
```

#### Example: Using the Secret in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-registry-pod
spec:
  containers:
    - name: my-container
      image: my-registry.io/my-image:latest
  imagePullSecrets:
    - name: my-registry-secret
```

### 7. **Storing and Managing Kubernetes Secrets Securely**
- **Use External Secret Stores**: Vault, AWS Secrets Manager, GCP Secret Manager.
- **Avoid Hardcoding Secrets in Manifests**.
- **Enable Encryption for Kubernetes Secrets**.
- **Restrict Access Using RBAC**.

#### Example: Encrypt Kubernetes Secrets at Rest
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64_ENCODED_SECRET>
      - identity: {}
```

#### Example: Using a Secret in an Environment Variable
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: my-container
      image: my-registry.io/my-image:latest
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

### 8. **Restrict Image Pull Sources**
- Limit image sources using **Kubernetes admission controllers**.
- Use **Namespace-level restrictions** to enforce registry policies.

#### Example: Restrict Image Sources with OPA
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata:
  name: allowed-repos
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    repos:
      - "my-registry.io/"
      - "gcr.io/"
```

---
## Best Practices for Image Security & Secret Management
✅ **Use minimal, trusted base images** to reduce attack surface.
✅ **Regularly scan container images** for vulnerabilities.
✅ **Sign and verify images** to ensure authenticity.
✅ **Restrict image sources** to approved registries.
✅ **Use private registries** and enable built-in scanning features.
✅ **Secure Docker builds** by avoiding secrets and privileged users.
✅ **Use Kubernetes Secrets securely and encrypt them at rest.**
✅ **Leverage external secret stores for sensitive credentials.**
✅ **Implement RBAC to restrict access to sensitive data.**

---
**Reference**: [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/overview/)
