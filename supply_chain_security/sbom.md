# Software Bill of Materials (SBOM)

## What is an SBOM?
A **Software Bill of Materials (SBOM)** is a machine-readable list of all components, libraries, and dependencies used in a software system. It helps organizations track vulnerabilities, ensure compliance, and improve software supply chain security.

## Why is SBOM Important for CKS?
 - **Identifies vulnerable dependencies** in container images.
 - **Enhances supply chain security** by tracking open-source components.
 - **Facilitates compliance** with frameworks like **SLSA, NIST SSDF, and CIS Benchmarks**.
 - **Improves transparency** in software dependencies.

---
## SBOM Formats
Kubernetes and cloud-native security tools support the following SBOM formats:
- **SPDX (Software Package Data Exchange)** – widely used in open-source compliance.
- **CycloneDX** – optimized for vulnerability tracking.
- **Syft JSON** – lightweight format used by `syft`.

---
## Generating SBOMs in Kubernetes

### 1. **Using Syft to Generate SBOMs**
[Syft](https://github.com/anchore/syft) is a popular SBOM tool for containers and Kubernetes.

#### Install Syft:
```sh
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh
```

#### Generate SBOM for a Container Image:
```sh
syft my-registry.io/my-image:latest -o spdx-json
```

#### Generate SBOM for a Kubernetes Pod:
```sh
kubectl exec -it <pod-name> -- syft -o cyclonedx-json /
```

### 2. **Using Trivy to Generate SBOMs**
[Trivy](https://aquasecurity.github.io/trivy/) can generate SBOMs along with vulnerability scanning.

#### Install Trivy:
```sh
brew install aquasecurity/trivy/trivy  # macOS
sudo apt install -y trivy  # Ubuntu/Debian
```

#### Generate SBOM in SPDX format:
```sh
trivy sbom --format spdx-json -o sbom.json my-registry.io/my-image:latest
```

### 3. **Using Kubernetes Native SBOM Tools**
- **kubectl-sbom** – generates SBOMs for running Pods.

#### Install kubectl-sbom:
```sh
kubectl krew install sbom
```

#### Generate SBOM for a Pod:
```sh
kubectl sbom <pod-name> --format spdx-json
```

---
## Storing and Verifying SBOMs
- Store SBOMs in a **secure artifact repository** (e.g., GitHub, GitLab, Amazon ECR, Google Artifact Registry).
- Use **in-toto Attestation** to verify SBOM authenticity.

#### Example: Attaching an SBOM to an OCI Image
```sh
cosign attach sbom --sbom sbom.json my-registry.io/my-image:latest
```

#### Example: Verifying an SBOM Signature
```sh
cosign verify-attestation --type sbom my-registry.io/my-image:latest
```

---
## Best Practices for SBOM in Kubernetes Security
✅ **Generate SBOMs for all container images** before deployment.
✅ **Store SBOMs securely** in an artifact registry.
✅ **Regularly scan SBOMs for vulnerabilities** using Trivy or Grype.
✅ **Verify SBOM signatures** to prevent tampering.
✅ **Use SBOM data** to enforce security policies in CI/CD pipelines.

---
**Reference**: [SBOM Best Practices](https://slsa.dev/)
