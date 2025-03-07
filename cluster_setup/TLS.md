# CKS Cheat Sheet: TLS in Kubernetes

## User Authentication with TLS

### 1. Generate a TLS Certificate for a User
```sh
openssl genrsa -out user-key.pem 2048
openssl req -new -key user-key.pem -out user.csr -subj "/CN=username/O=group"
openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user.crt -days 365
```

### 2. Configure kubectl to Use the Certificate
```sh
kubectl config set-credentials username \
    --client-certificate=user.crt \
    --client-key=user-key.pem
kubectl config set-context user-context --cluster=kubernetes-cluster --user=username
kubectl config use-context user-context
```

### 3. Verify User Authentication
```sh
kubectl auth can-i get pods --as=username
```

---

## Securing Kubernetes API with TLS

### 1. Configure API Server with TLS
- Ensure API server is running with `--tls-cert-file` and `--tls-private-key-file` flags.
- Verify flags:
```sh
ps aux | grep kube-apiserver | grep tls
```

### 2. Check API Server Certificate Details
```sh
kubectl get csr
kubectl describe csr <csr-name>
```

### 3. Manually Approve a CSR
```sh
kubectl certificate approve <csr-name>
```

### 4. Generate a New TLS Certificate for the API Server
```sh
openssl req -new -key apiserver-key.pem -out apiserver.csr -subj "/CN=kube-apiserver"
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -days 365
```

### 5. Verify API Server TLS Configuration
```sh
curl -v --cacert ca.crt https://kubernetes.default.svc:6443
```
## References

  - [Kubernetes cert checker](https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools)

