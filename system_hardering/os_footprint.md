# CKS Cheat Sheet: Minimize Host OS Footprint

## 1. Use Minimalist Operating Systems
- Use lightweight, container-optimized OS options:
  - **Flatcar Container Linux**
  - **Ubuntu Core**
  - **Alpine Linux**
  - **Google Container-Optimized OS (COS)**
  - **Red Hat CoreOS**

## 2. Remove Unnecessary Packages and Services
- List installed packages:
  ```bash
  dpkg --list   # Debian-based
  rpm -qa       # RHEL-based
  ```
- Remove unnecessary packages:
  ```bash
  sudo apt remove --purge package-name -y  # Debian-based
  sudo yum remove package-name -y         # RHEL-based
  ```
- Disable unused services:
  ```bash
  sudo systemctl disable --now service-name
  ```
- List and stop unneeded services:
  ```bash
  sudo systemctl list-units --type=service --state=running
  ```

## 3. Harden Kernel and OS Security
- Enable automatic security updates:
  ```bash
  sudo apt install unattended-upgrades -y
  sudo dpkg-reconfigure unattended-upgrades
  ```
- Restrict kernel features using sysctl:
  ```bash
  sudo nano /etc/sysctl.conf
  ```
  Add:
  ```bash
  net.ipv4.conf.all.accept_redirects = 0
  net.ipv4.conf.all.send_redirects = 0
  kernel.unprivileged_bpf_disabled = 1
  ```
  Apply changes:
  ```bash
  sudo sysctl -p
  ```

## 4. Use Immutable Infrastructure
- Deploy hosts as immutable images (e.g., **Golden Images**, **Immutable Servers**).
- Use **OSTree**-based systems (e.g., Fedora Silverblue, Red Hat CoreOS).
- Avoid SSH access to production hosts.

## 5. Restrict Root Access and User Privileges
- Lock the root account:
  ```bash
  sudo passwd -l root
  ```
- Create a restricted user with minimal privileges:
  ```bash
  sudo adduser secureuser
  sudo usermod -aG sudo secureuser
  ```
- Use **sudoers** file to limit commands:
  ```bash
  sudo visudo
  ```
  Add:
  ```bash
  secureuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
  ```

## 6. Monitor and Audit OS Footprint
- Use **auditd** to track system changes:
  ```bash
  sudo auditctl -w /etc/passwd -p wa -k user_modifications
  sudo auditctl -w /usr/bin/docker -p x -k docker_execution
  ```
- Check audit logs:
  ```bash
  sudo ausearch -k user_modifications
  sudo ausearch -k docker_execution
  ```

