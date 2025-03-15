# Seccomp Cheat Sheet for CKS Kubernetes Exam

## 1. Overview of Seccomp
- **Seccomp (Secure Computing Mode)** is a Linux kernel feature that restricts the system calls a process can make.
- Helps reduce the attack surface by limiting access to unnecessary syscalls.
- Used to enhance container security in Kubernetes.

## 2. Checking Seccomp Support
```sh
grep SECCOMP /boot/config-$(uname -r)
```
- Ensures seccomp is enabled in the kernel.

## 3. Seccomp Profiles
- Located in `/var/lib/kubelet/seccomp/` or specified in custom paths.
- Common profiles:
  - `runtime/default` (Docker/Kubernetes default)
  - `unconfined` (No restrictions)
  - Custom profiles (JSON format)

## 4. Applying Seccomp to Kubernetes Pods
- Seccomp can be enforced via **annotations** (deprecated) or **securityContext** in Pod specifications.
- Example using **annotations** (deprecated in Kubernetes 1.27+):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-pod
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: localhost/my-seccomp-profile.json
spec:
  containers:
  - name: my-container
    image: nginx
```

- Example using **securityContext** (preferred method):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost  # Or 'RuntimeDefault'
      localhostProfile: my-seccomp-profile.json
  containers:
  - name: my-container
    image: nginx
```

## 5. Creating a Custom Seccomp Profile
1. Create a JSON profile:

```sh
sudo nano /var/lib/kubelet/seccomp/my-seccomp-profile.json
```

2. Define rules (example profile):
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["exit", "read", "write", "fstat"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

3. Apply the profile to a pod.

## 6. Listing Available Seccomp Profiles
```sh
ls /var/lib/kubelet/seccomp/
```

## 7. Testing and Debugging Seccomp
- **Check running container's seccomp status:**
```sh
docker inspect --format '{{ .HostConfig.SecurityOpt }}' <container_id>
```
- **Test blocked syscalls:**
```sh
strace -c -p <pid>
```
- **View seccomp violations:**
```sh
journalctl -k | grep seccomp
```

## 8. Disabling Seccomp for Debugging (Not Recommended for Production)
- Set `seccompProfile.type: Unconfined` in pod security context.

## 9. Exam Tips
- Know how to **apply Seccomp profiles** using annotations and `securityContext`.
- Be familiar with **default, unconfined, and custom profiles**.
- Understand how to **troubleshoot Seccomp violations**.
- Be comfortable with JSON profile syntax and `seccompProfile` settings.

---

