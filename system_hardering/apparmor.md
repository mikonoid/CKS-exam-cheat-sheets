# AppArmor Cheat Sheet for CKS Kubernetes Exam

## 1. Overview of AppArmor
- **AppArmor (Application Armor)** is a Linux security module that restricts programs' capabilities using per-program profiles.
- Works in **enforce** mode (blocking unauthorized actions) or **complain** mode (logging unauthorized actions without blocking).
- Used to strengthen container security in Kubernetes.

## 2. Checking AppArmor Status
```sh
sudo apparmor_status
```
- Displays loaded AppArmor profiles and their modes.

## 3. AppArmor Profiles
- Located in `/etc/apparmor.d/`.
- Profiles define rules for what a process can do (e.g., file access, networking, system calls).

## 4. Enforcing AppArmor on Kubernetes Pods
- AppArmor can be enforced via **annotations** in Pod specifications.
- Example of enabling an AppArmor profile for a container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/my-container: localhost/my-profile
spec:
  containers:
  - name: my-container
    image: nginx
```
- `localhost/my-profile` refers to an AppArmor profile preloaded on the node.

## 5. Default Profiles in Kubernetes Nodes
- Ubuntu-based Kubernetes nodes typically include the `docker-default` profile.
- Default profile example: `/etc/apparmor.d/docker-default`.
- Can be customized for specific workloads.

## 6. Listing AppArmor Profiles
```sh
ls /etc/apparmor.d/
```
- Displays available profiles.

## 7. Creating a Custom AppArmor Profile
1. Create a profile in `/etc/apparmor.d/`:

```sh
sudo nano /etc/apparmor.d/my-profile
```

2. Define rules (example profile):
```plaintext
#include <tunables/global>
profile my-profile flags=(attach_disconnected,mediate_deleted) {
  capability net_bind_service,
  network,
  file,
  /bin/bash ix,
  /usr/bin/curl px,
}
```

3. Load the profile:
```sh
sudo apparmor_parser -r -W /etc/apparmor.d/my-profile
```

## 8. Enforcing or Setting AppArmor Mode
```sh
sudo aa-enforce /etc/apparmor.d/my-profile  # Enforce mode
sudo aa-complain /etc/apparmor.d/my-profile  # Complain mode
```

## 9. Troubleshooting AppArmor Issues
- **View logs for AppArmor violations:**
```sh
sudo dmesg | grep -i apparmor
journalctl -k | grep apparmor
```
- **Disable AppArmor for debugging (not recommended for production):**
```sh
sudo systemctl disable apparmor
sudo systemctl stop apparmor
```

## 10. Exam Tips
- Know how to **enable, disable, and enforce AppArmor profiles** on Kubernetes Pods.
- Be familiar with **default profiles** and **custom AppArmor profiles**.
- Understand how to **troubleshoot AppArmor denials**.
- Be comfortable with `apparmor_parser`, `aa-enforce`, and `aa-complain` commands.

---

