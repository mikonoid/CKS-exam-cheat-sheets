# Trivy

## What is Trivy?
[Trivy](https://aquasecurity.github.io/trivy/) is an open-source vulnerability scanner for containers, Kubernetes, and infrastructure-as-code (IaC). It helps detect security risks and misconfigurations before deployment.

## Why is Trivy Important for CKS?
✅ **Scans container images** for vulnerabilities.
✅ **Checks Kubernetes configurations** for security issues.
✅ **Integrates with CI/CD pipelines** to prevent insecure deployments.
✅ **Supports multiple scanning targets**, including filesystems, Git repositories, and cloud services.

---
## Installing Trivy

### Install on macOS
```sh
brew install aquasecurity/trivy
```

### Install on Linux
```sh
sudo apt install -y trivy  # Debian/Ubuntu
```

### Install on Kubernetes (Helm)
```sh
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
helm install trivy aquasecurity/trivy-operator --namespace trivy-system --create-namespace
```

---
## Scanning Containers & Kubernetes Resources

### 1. **Scanning a Container Image**
```sh
trivy image my-registry.io/my-image:latest
```

### 2. **Scanning a Kubernetes Cluster**
```sh
trivy k8s --report summary
```

### 3. **Scanning Kubernetes Resources**
```sh
trivy k8s cluster
trivy k8s node
trivy k8s pod --namespace default
```

---
## Scanning Filesystems & Repositories

### 4. **Scanning a Filesystem**
```sh
trivy fs /path/to/project
```

### 5. **Scanning Infrastructure-as-Code (IaC) Files**
```sh
trivy config /path/to/kubernetes-manifests
```

### 6. **Scanning a Git Repository**
```sh
trivy repo https://github.com/example/repo.git
```

---
## Scanning with Different Output Formats

### JSON Output
```sh
trivy image --format json -o results.json my-registry.io/my-image:latest
```

### Table Output (Default)
```sh
trivy image --format table my-registry.io/my-image:latest
```

### SPDX SBOM Format
```sh
trivy sbom --format spdx-json my-registry.io/my-image:latest
```

---
## CI/CD Integration

### GitHub Actions Example
```yaml
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Scan Image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: "my-registry.io/my-image:latest"
          format: "table"
          exit-code: 1
          severity: "CRITICAL,HIGH"
```

### GitLab CI/CD Example
```yaml
security_scan:
  image: aquasec/trivy:latest
  script:
    - trivy image my-registry.io/my-image:latest
```

---
## Best Practices for Using Trivy in Kubernetes Security
✅ **Regularly scan container images** before deployment.
✅ **Integrate Trivy into CI/CD pipelines** to detect vulnerabilities early.
✅ **Scan Kubernetes resources** to identify misconfigurations.
✅ **Use different severity levels** to prioritize security fixes.
✅ **Export SBOMs** and store them securely for audit purposes.

---
**Reference**: [Trivy Documentation](https://aquasecurity.github.io/trivy/)
