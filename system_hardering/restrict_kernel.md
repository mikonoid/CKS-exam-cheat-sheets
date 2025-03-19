# CKS Cheat Sheet: Restrict Kernel Modules

## 1. Disable Unnecessary Kernel Modules
- List loaded kernel modules:
  ```bash
  lsmod
  ```
- Disable loading specific modules by adding them to `/etc/modprobe.d/blacklist.conf`:
  ```bash
  echo 'blacklist usb-storage' | sudo tee -a /etc/modprobe.d/blacklist.conf
  ```
- Apply changes:
  ```bash
  sudo update-initramfs -u
  ```

## 2. Prevent Kernel Module Loading at Runtime
- Restrict module loading:
  ```bash
  sudo sysctl -w kernel.modules_disabled=1
  ```
- Make it persistent:
  ```bash
  echo 'kernel.modules_disabled=1' | sudo tee -a /etc/sysctl.conf
  sudo sysctl -p
  ```

## 3. Use `kmod` to Manage Kernel Modules Securely
- Remove an unwanted module:
  ```bash
  sudo modprobe -r <module_name>
  ```
- Check dependencies before removing:
  ```bash
  sudo lsmod | grep <module_name>
  ```

## 4. Restrict Module Loading Using SELinux or AppArmor
- For SELinux, enforce module restrictions:
  ```bash
  sudo semanage boolean -m --on secure_mode_insmod
  ```
- For AppArmor, create a policy to prevent module loading.

## 5. Use Kubernetes Security Contexts
- Prevent privileged containers from loading kernel modules:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: restricted-pod
  spec:
    containers:
    - name: secure-container
      image: busybox
      securityContext:
        privileged: false
  ```

## 6. Monitor Kernel Module Loading Events
- Use `auditd` to track module loads:
  ```bash
  sudo auditctl -w /sbin/insmod -p x -k module_load
  ```
- Check audit logs:
  ```bash
  sudo ausearch -k module_load
  ```


