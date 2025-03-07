# CKS Cheat Sheet: CIS Benchmarks & kube-bench

## 1. What is CIS?
The **Center for Internet Security (CIS)** is a nonprofit organization that leverages a global IT community to develop best practices for securing systems against cyber threats.

### CIS Benchmarks
- A set of globally recognized best practices to secure IT systems.
- Developed through consensus by cybersecurity experts.
- Covers operating systems, cloud providers, containers, and Kubernetes.
- More details: [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)

---

## 2. CIS Kubernetes Benchmark Overview
The **CIS Kubernetes Benchmark** provides security best practices to harden Kubernetes clusters.

### Key Security Areas:
1. **Control Plane Security**: API server, Controller Manager, Scheduler
2. **ETCD Security**: Data encryption and access control
3. **RBAC & Authentication**: Secure user and service access
4. **Logging & Auditing**: Track API actions and detect security threats
5. **Network Policies**: Restrict unnecessary communication
6. **Worker Node Security**: Secure kubelet and container runtime

---

## 3. What is kube-bench?
**kube-bench** is an open-source tool that checks Kubernetes clusters for compliance with the CIS Kubernetes Benchmark.

### Features:
- Scans cluster configurations against CIS benchmarks.
- Reports compliance status and potential security risks.
- Supports different Kubernetes versions and distributions.
- Does not enforce fixes but provides recommendations.

### Install and Run kube-bench
#### Install kube-bench:
```sh
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
```

#### Run kube-bench:
```sh
kubectl logs -f <kube-bench-pod>
```

#### Run kube-bench on a node:
```sh
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -v /etc:/etc:ro -v /usr/bin:/usr/bin:ro -v /var:/var:ro \
  aquasec/kube-bench:latest
```

---

## 4. Fixing Common CIS Benchmark Issues

### 🔹 API Server Hardening
**Issue:** Insecure port enabled
**Fix:**
```sh
--insecure-port=0
```

**Issue:** Anonymous access allowed
**Fix:**
```sh
--anonymous-auth=false
```

**Issue:** Audit logging not enabled
**Fix:**
```sh
--audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

---

### 🔹 ETCD Security
**Issue:** ETCD data is not encrypted
**Fix:**
```sh
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

**Issue:** Unrestricted access to ETCD
**Fix:**
```sh
--client-cert-auth=true --peer-client-cert-auth=true
```

---

### 🔹 Worker Node Security
**Issue:** Kubelet allows anonymous requests
**Fix:**
```sh
--anonymous-auth=false
```

**Issue:** Default seccomp profile not enabled
**Fix:**
```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```






