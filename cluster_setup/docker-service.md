# CKS Exam Cheat Sheet: Docker Security

## 1. Docker Service Configuration for CKS

### Securing Docker Daemon
- Disable unauthenticated access:
  ```sh
  sudo systemctl disable docker.socket
  ```
- Enforce TLS for Docker daemon:
  ```sh
  sudo dockerd --tlsverify \
    --tlscacert=/etc/docker/ca.pem \
    --tlscert=/etc/docker/server-cert.pem \
    --tlskey=/etc/docker/server-key.pem
  ```
- Configure daemon options in `/etc/docker/daemon.json`:
  ```json
  {
    "tls": true,
    "tlsverify": true,
    "tlscacert": "/etc/docker/ca.pem",
    "tlscert": "/etc/docker/server-cert.pem",
    "tlskey": "/etc/docker/server-key.pem",
    "userns-remap": "default",
    "no-new-privileges": true
  }
  ```
- Restart Docker to apply changes:
  ```sh
  sudo systemctl daemon-reload
  sudo systemctl restart docker
  ```

## 2. Docker - Securing the Daemon

### Limit Root Privileges
- Enable user namespace remapping:
  ```sh
  sudo usermod -aG docker <username>
  echo '{"userns-remap": "default"}' | sudo tee -a /etc/docker/daemon.json
  ```
- Disable privileged mode for containers
- Use seccomp profiles:
  ```sh
  --security-opt seccomp=default.json
  ```

### Network Security
- Restrict inter-container communication:
  ```json
  {
    "icc": false
  }
  ```
- Limit container network exposure:
  ```sh
  docker network create --internal secure-net
  ```
- Use `iptables` rules to control container traffic

### Logging & Monitoring
- Enable audit logging:
  ```sh
  sudo auditctl -w /usr/bin/docker -k docker
  ```
- Monitor Docker logs:
  ```sh
  sudo journalctl -u docker -f
  ```
- Use Falco for runtime security monitoring

## 3. CIS Docker Benchmark Recommendations
- Ensure `/var/lib/docker` has correct permissions
- Restrict access to Docker daemon socket (`/var/run/docker.sock`)
- Enable live-restore:
  ```json
  {
    "live-restore": true
  }
  ```
- Set resource limits:
  ```sh
  --memory=512m --cpu-shares=1024
  ```

## 4. Best Practices
- Use minimal base images (e.g., `distroless`, `alpine`)
- Regularly scan images for vulnerabilities:
  ```sh
  docker scan <image>
  ```
- Keep Docker engine up to date
- Implement least privilege policies

## References
- [Docker Official Docs](https://docs.docker.com/)
- [CIS Benchmark for Docker](https://www.cisecurity.org/benchmark/docker)

