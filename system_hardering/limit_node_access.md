# CKS Cheat Sheet: Limiting Node Access (Linux Users & Access)

## 1. **Restrict SSH Access**
- **Disable root login** via SSH:
  
  ```bash
  sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  ```
- **Allow only specific users to SSH**:
  
  ```bash
  echo 'AllowUsers user1 user2' | sudo tee -a /etc/ssh/sshd_config
  sudo systemctl restart sshd
  ```
- Use **key-based authentication** and **disable password authentication**:
  
  ```bash
  sudo sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  ```

## 2. **Use Sudoers for Privilege Restriction**
- **Grant least privilege** to users using sudo:
  
  ```bash
  sudo visudo
  ```
  
  Add:
  
  ```
  user1 ALL=(ALL) NOPASSWD: /usr/bin/kubectl
  ```
  
  This allows `user1` to use `kubectl` without full root access.

## 3. **Restrict User Access to Nodes**
- **Create dedicated user groups** for Kubernetes administration:
  
  ```bash
  sudo groupadd k8s-admins
  sudo usermod -aG k8s-admins user1
  ```
- **Use PAM to limit login access**:
  
  ```bash
  echo 'auth required pam_listfile.so onerr=fail item=user sense=allow file=/etc/allowed_users' | sudo tee -a /etc/pam.d/sshd
  echo 'user1' | sudo tee -a /etc/allowed_users
  ```

## 4. **Restrict Root User Access**
- **Lock the root account:**
  
  ```bash
  sudo passwd -l root
  ```
- **Limit who can use `su` to switch to root:**
  
  ```bash
  sudo groupadd wheel
  sudo usermod -aG wheel user1
  echo 'auth required pam_wheel.so use_uid' | sudo tee -a /etc/pam.d/su
  ```

## 5. **Limit File & Directory Permissions**
- **Ensure only necessary users can access Kubernetes configuration files**:
  
  ```bash
  sudo chmod 600 /etc/kubernetes/admin.conf
  sudo chown root:k8s-admins /etc/kubernetes/admin.conf
  ```
- **Secure Kubernetes certificates**:
  
  ```bash
  sudo chmod 600 /etc/kubernetes/pki/*
  sudo chown root:k8s-admins /etc/kubernetes/pki/*
  ```

## 6. **Monitor and Audit User Access**
- **Enable system auditing with auditd**:
  
  ```bash
  sudo apt install auditd -y  # Ubuntu/Debian
  sudo yum install audit -y   # RHEL/CentOS
  sudo systemctl enable auditd --now
  ```
- **Log sudo commands**:
  
  ```bash
  echo 'Defaults logfile=/var/log/sudo.log' | sudo tee -a /etc/sudoers
  ```
- **Track SSH login attempts**:
  
  ```bash
  sudo cat /var/log/auth.log | grep 'sshd'
  ```

## 7. **Use SELinux or AppArmor for Access Control**
- **Enable SELinux in enforcing mode:**
  
  ```bash
  sudo setenforce 1
  sudo sed -i 's/^SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config
  ```
- **Enable AppArmor for Kubelet**:
  
  ```bash
  sudo aa-enforce /etc/apparmor.d/usr.sbin.kubelet
  ```

## 8. **Restrict User Cron Jobs and Background Processes**
- **Disable cron jobs for untrusted users**:
  
  ```bash
  sudo echo 'user1' > /etc/cron.deny
  ```
- **Limit background process execution**:
  
  ```bash
  sudo echo 'user1 hard nproc 50' >> /etc/security/limits.conf
  ```
