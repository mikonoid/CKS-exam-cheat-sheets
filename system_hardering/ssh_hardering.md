# CKS Cheat Sheet: SSH Hardening

## 1. **Disable Root Login**
Prevent direct root access via SSH:

```bash
sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## 2. **Use Key-Based Authentication**
Disable password authentication to enforce SSH key usage:

```bash
sudo sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

Ensure users have SSH keys:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vi ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## 3. **Limit SSH Access to Specific Users**
Define which users can SSH into the system:

```bash
echo 'AllowUsers user1 user2' | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## 4. **Restrict SSH Access by IP Address**
Allow SSH access only from trusted IP ranges:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -s <trusted-ip> -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

Or use UFW (Uncomplicated Firewall):

```bash
sudo ufw allow from <trusted-ip> to any port 22
```

## 5. **Change the Default SSH Port**
Modify the SSH port to reduce brute-force attacks:

```bash
sudo sed -i 's/^#Port 22/Port 2222/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

Ensure the new port is allowed in the firewall:

```bash
sudo ufw allow 2222/tcp
```

## 6. **Use SSH Banner and Warning Messages**
Display a security warning before authentication:

```bash
echo 'Unauthorized access is prohibited!' | sudo tee /etc/issue.net
sudo sed -i 's/^#Banner none/Banner \/etc\/issue.net/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## 7. **Limit Connection Attempts**
Prevent brute-force attacks by limiting authentication retries:

```bash
sudo sed -i 's/^#MaxAuthTries 6/MaxAuthTries 3/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## 8. **Enable Two-Factor Authentication (2FA)**
Install Google Authenticator:

```bash
sudo apt install libpam-google-authenticator -y
```

Configure PAM to enforce 2FA:

```bash
echo "auth required pam_google_authenticator.so" | sudo tee -a /etc/pam.d/sshd
```

Enable ChallengeResponseAuthentication:

```bash
sudo sed -i 's/^ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

## 9. **Use Fail2Ban for SSH Protection**
Install Fail2Ban:

```bash
sudo apt install fail2ban -y
```

Configure SSH jail settings:

```bash
sudo tee /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
port = 22
maxretry = 3
findtime = 600
bantime = 3600
EOF
```

Restart Fail2Ban:

```bash
sudo systemctl restart fail2ban
```

## 10. **Monitor SSH Access Logs**
Track SSH login attempts and failures:

```bash
tail -f /var/log/auth.log | grep sshd
