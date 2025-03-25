# Supply Chain Security Overview

## What is Supply Chain Security in Kubernetes?
Supply Chain Security in Kubernetes ensures the integrity, authenticity, and security of software components across the entire development lifecycle—**from source code to deployment**.

## Why is Supply Chain Security Important for CKS?
- Prevents **supply chain attacks** like dependency poisoning and build system compromises.
- Ensures **image integrity** and prevents unauthorized modifications.
- Verifies that only **trusted artifacts** are deployed to production.
- Helps comply with security standards like **SLSA, NIST SSDF, and CIS Benchmarks**.

---
## Key Components of Kubernetes Supply Chain Security

### 1. **Source Code Security**
- Use **code signing (e.g., Sigstore, GPG)** to verify commit integrity.
- Implement **code scanning tools** (e.g., Snyk, Trivy, Checkov) to detect vulnerabilities.
- Enforce **branch protection and code reviews** in Git repositories.
- Enable **developer access control** with IAM and MFA.

#### Example: Scan for vulnerabilities using Trivy
```sh
trivy fs .
```

### 2. **Build Security**
- Use **isolated and hardened CI/CD environments** (e.g., GitHub Actions, GitLab CI/CD, Tekton).
- Sign and verify builds using **Sigstore Cosign**.
- Detect and block malicious dependencies with **Dependency Track or Snyk**.

#### Example: Signing a container image with Cosign
```sh
cosign sign --key cosign.key my-registry.io/my-image:latest
```

#### Example: Verifying an image signature
```sh
cosign verify my-registry.io/my-image:latest --key cosign.pub
```

### 3. **Artifact Security**
- Store and distribute images securely via **private container registries** (e.g., Amazon ECR, Google Artifact Registry).
- Use **immutable tags** and avoid using `latest`.
- Scan images for vulnerabilities before deployment using **Trivy or Clair**.

#### Example: Scan an image for vulnerabilities
```sh
trivy image my-registry.io/my-image:latest
```

### 4. **Deployment Security**
- Enforce **Admission Controllers** (e.g., OPA Gatekeeper, Kyverno) to block untrusted images.
- Use **Pod Security Standards (PSS)** to prevent privilege escalation.
- Require **only signed images** for deployments with Sigstore or Notary.

#### Example: Kyverno policy to allow only signed images
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signatures
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-signature
      match:
        resources:
          kinds:
            - Pod
      verifyImages:
        - image: "my-registry.io/*"
          key: cosign.pub
```

### 5. **Runtime Security**
- Use **Runtime Security tools** (e.g., Falco, Tracee) to detect anomalous behavior.
- Enforce **Least Privilege Access** (e.g., dropping unnecessary capabilities in Pods).
- Monitor and log Kubernetes API activity using **Audit Logs** and **SIEM solutions**.

#### Example: Monitor unexpected network activity using Falco
```sh
falco --list
```

---
## Best Practices for Supply Chain Security
✅ **Secure the source code** with version control best practices and scanning tools.
✅ **Harden CI/CD pipelines** to prevent tampering.
✅ **Sign and verify artifacts** before deployment.
✅ **Enforce admission controls** to block untrusted containers.
✅ **Monitor runtime security** using tools like Falco and Audit Logs.

---
**Reference**: [Supply Chain Security Best Practices](https://slsa.dev)
