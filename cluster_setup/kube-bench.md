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

#### Install kube-bench:
```sh
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
```

#### Run kube-bench on controlplane:
```sh
./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml 
```

---

## 4. Fixing Exampl CIS Benchmark Issues

### Example output

```
[FAIL] 1.2.16 Ensure that the admission control plugin PodSecurityPolicy is set (Automated)
[PASS] 1.2.17 Ensure that the admission control plugin NodeRestriction is set (Automated)
[PASS] 1.2.18 Ensure that the --insecure-bind-address argument is not set (Automated)
[FAIL] 1.2.19 Ensure that the --insecure-port argument is set to 0 (Automated)
[PASS] 1.2.20 Ensure that the --secure-port argument is not set to 0 (Automated)
[PASS] 1.2.21 Ensure that the --profiling argument is set to false (Automated)
[FAIL] 1.2.22 Ensure that the --audit-log-path argument is set (Automated)
[FAIL] 1.2.23 Ensure that the --audit-log-maxage argument is set to 30 or as appropriate (Automated)
[FAIL] 1.2.24 Ensure that the --audit-log-maxbackup argument is set to 10 or as appropriate (Automated)
[FAIL] 1.2.25 Ensure that the --audit-log-maxsize argument is set to 100 or as appropriate (Automated)
[PASS] 1.2.26 Ensure that the --request-timeout argument is set as appropriate (Automated)
```

### API Server Hardening
**Issue:** Insecure port enabled
**Fix:**
Add to /etc/kubernetes/manifests/kube-apiserver.yaml

```sh
--insecure-port=0
```

**Issue:** Profiling enabled
**Fix:**
Add to /etc/kubernetes/manifests/kube-apiserver.yaml

```sh
--profiling=false
```






