# CKS Cheat Sheet: Linux Privilege Escalation Prevention

## 1. Identify and Remove Unnecessary SUID/SGID Binaries
- List all files with the SUID bit set:
  ```bash
  find / -perm -4000 -type f 2>/dev/null
  ```
- List all files with the SGID bit set:
  ```bash
  find / -perm -2000 -type f 2>/dev/null
  ```
- Remove SUID/SGID permissions if not needed:
  ```bash
  sudo chmod u-s /path/to/binary
  sudo chmod g-s /path/to/binary
  ```

## 2. Restrict `sudo` Access
- List users with `sudo` privileges:
  ```bash
  getent group sudo
  ```
- Edit `/etc/sudoers` securely using:
  ```bash
  sudo visudo
  ```
- Remove unnecessary sudo access:
  ```bash
  sudo deluser username sudo
  ```

## 3. Disable Root SSH Access
- Edit the SSH configuration file:
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
- Set:
  ```bash
  PermitRootLogin no
  ```
- Restart SSH service:
  ```bash
  sudo systemctl restart sshd
  ```

## 4. Limit `su` Command Usage
- Restrict `su` to a specific group (e.g., `wheel`):
  ```bash
  sudo nano /etc/pam.d/su
  ```
- Uncomment the line:
  ```bash
  auth required pam_wheel.so use_uid
  ```
- Add allowed users to the `wheel` group:
  ```bash
  sudo usermod -aG wheel username
  ```

## 5. Protect Sensitive Files and Directories
- Set correct permissions on sensitive files:
  ```bash
  sudo chmod 640 /etc/shadow
  sudo chmod 644 /etc/passwd
  ```

## 6. Use AppArmor/SELinux to Restrict Privilege Escalation
- Enforce AppArmor profiles:
  ```bash
  sudo aa-enforce /etc/apparmor.d/*
  ```
- Check SELinux mode:
  ```bash
  getenforce
  ```
- Set SELinux to Enforcing mode:
  ```bash
  sudo setenforce 1
  ```

## 7. Monitor Privilege Escalation Attempts
- Use `auditd` to track sudo access:
  ```bash
  sudo auditctl -w /etc/sudoers -p wa -k sudo_access
  ```
- View logs:
  ```bash
  sudo ausearch -k sudo_access
  ```

