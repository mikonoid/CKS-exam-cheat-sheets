# CKS Exam Cheat Sheet: Kubelet Security

## 1. Kubelet Authentication & Authorization
### Kubelet API Authentication
- Uses client certificates (`--client-ca-file`)
- Token authentication (`--authentication-token-webhook`)
- Disable anonymous access (`--anonymous-auth=false`)

### Kubelet Authorization
- Always use `--authorization-mode=Webhook`
- Avoid `AlwaysAllow` mode (insecure!)

## 2. Restrict Kubelet API Access
- Limit CIDR range using `--bind-address=127.0.0.1`
- Restrict insecure port (`--port=0`)
- Secure TLS (`--tls-cert-file` & `--tls-private-key-file`)
- Use Network Policies to limit kubelet API access

## 3. Pod Security & Kubelet Hardening
- Disable `--allow-privileged` (`--allow-privileged=false`)
- Restrict HostPath volumes using PSP/PSS
- Use Seccomp (`seccompProfile: RuntimeDefault`)
- Use AppArmor for process-level security
- Enforce least privilege with RBAC

## 4. Kubelet Secure Communication
- Always enable TLS (`--tls-cert-file`, `--tls-private-key-file`)
- Enable and validate certificates (`--client-ca-file`)
- Use kubelet server certificate rotation (`--rotate-server-certificates`)
- Secure etcd communication (`--etcd-certfile`, `--etcd-keyfile`)

## 5. Logging & Auditing
- Enable kubelet event logging (`--event-qps=5`)
- Use audit logging (`--audit-log-path` & `--audit-log-maxsize`)
- Review logs regularly (`journalctl -u kubelet`)

## 6. CIS Benchmark Recommendations
- Ensure `anonymous-auth=false`
- Enable `--protect-kernel-defaults`
- Set `--read-only-port=0` to disable unauthenticated access
- Ensure `--make-iptables-util-chains=true`

## 7. Best Practices
- Monitor Kubelet API access with Prometheus or Falco
- Regularly patch kubelet to avoid vulnerabilities
- Limit access to `/var/lib/kubelet` on worker nodes

## 8. Kubelet Configuration & Inspection
### Where to Find Kubelet Configuration
- Default locations:
  - `/var/lib/kubelet/config.yaml`
  - `/etc/kubernetes/kubelet.conf`
  - `/etc/systemd/system/kubelet.service.d/`

### How to Inspect Kubelet Configuration
- Check active config:
  ```sh
  sudo cat /var/lib/kubelet/config.yaml
  ```
- View systemd service:
  ```sh
  systemctl cat kubelet
  ```
- Check running arguments:
  ```sh
  ps -ef | grep kubelet
  ```

### How to Modify and Reload Kubelet Configuration
- Edit the kubelet config file:
  ```sh
  sudo vi /var/lib/kubelet/config.yaml
  ```
- Validate changes before applying:
  ```sh
  kubelet --config=/var/lib/kubelet/config.yaml --help
  ```
- Restart the kubelet service to apply changes:
  ```sh
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet
  ```
- Check kubelet logs for issues:
  ```sh
  journalctl -u kubelet -f
  ```

### References
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [CIS Benchmark for Kubernetes](https://www.cisecurity.org/benchmark/kubernetes)

