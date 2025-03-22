# gVisor

## What is gVisor?
gVisor is a user-space sandboxing runtime for containers that provides additional isolation between workloads and the host kernel.

## Why Use gVisor in Kubernetes?
- Reduces the attack surface by limiting kernel syscall access.
- Provides better isolation than traditional Linux namespaces.
- Protects against container escape vulnerabilities.

## Installing gVisor
### Install gVisor runtime
```sh
curl -fsSL https://storage.googleapis.com/gvisor/releases/release/latest/x86_64/runsc -o runsc
chmod +x runsc
sudo mv runsc /usr/local/bin/
```

### Install gVisor kernel module (optional)
```sh
curl -fsSL https://storage.googleapis.com/gvisor/releases/release/latest/x86_64/runsc-kernel -o runsc-kernel
chmod +x runsc-kernel
sudo mv runsc-kernel /usr/local/bin/
```

## Configuring gVisor with containerd
### Enable `runsc` in `containerd`:
Modify `/etc/containerd/config.toml`:
```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
runtime_type = "io.containerd.runsc.v1"
```
Restart containerd:
```sh
sudo systemctl restart containerd
```

## Configuring gVisor in Kubernetes
### Install `runsc` as a runtime in Kubernetes
Create a **RuntimeClass**:
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```

### Run a Pod with gVisor
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gvisor-test
spec:
  runtimeClassName: gvisor
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
```

## Verifying gVisor Usage
- Check if gVisor is running:
  ```sh
  runsc --version
  ```
- Ensure the runtime is configured correctly:
  ```sh
  kubectl get runtimeclass
  ```
- Check container runtime:
  ```sh
  kubectl get pod gvisor-test -o jsonpath='{.spec.runtimeClassName}'
  ```

## Security Benefits of gVisor
- Blocks **privileged** container access to host syscalls.
- Limits **ptrace-based** attacks.
- Prevents **syscall-based** exploits using a virtualized syscall layer.

## Best Practices for gVisor in Kubernetes
- Use gVisor for **multi-tenant workloads**.
- Restrict privileged containers when using gVisor.
- Test application compatibility with gVisor since not all syscalls are supported.
- Monitor gVisor logs for debugging security issues:
  ```sh
  journalctl -u containerd | grep runsc
  ```

---
**Reference**: [gVisor Documentation](https://gvisor.dev/docs/)
