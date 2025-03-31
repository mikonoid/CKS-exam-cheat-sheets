# CKS exam preparation cheat sheets
Preparation for CKS exam (Certified Kubernetes Security Specialist)

Topics:

* CKS exam introduction
* Custer Setup and Hardering
* System Hardering
* Minimize microservice Vulnerabilities
* Supply chain security
* Monitoring, Logging and Runtime security
* Exam tips

## CKS exam introduction

 - [Introduction](cluster_setup/introduction.md)

## Custer Setup and Hardering

 - [CIS benchmark and kube-bench](cluster_setup/kube-bench.md)
 - [Service Accounts](cluster_setup/sa.md)
 - [TLS in Kubernetes](cluster_setup/TLS.md)
 - [API groups](cluster_setup/apigroups.md)
 - [Authorization](cluster_setup/autorisation.md)
 - [RBAC](cluster_setup/rbac.md)
 - [Cluster upgrade process](cluster_setup/upgrade.md)
 - [Kubelet](cluster_setup/Kubelet.md)
 - [Network policies](cluster_setup/NetworkPolicy.md)
 - [Ingress](cluster_setup/ingress.md.md)
 - [Docker service security](cluster_setup/docker-service.md)
 - [Kubectl Proxy & Port Forward](cluster_setup/kubectl-forward.md)
 - [Auditing](cluster_setup/auditing.md)

## System Hardering

 - [Minimize host OS footprint](system_hardering/os_footprint.md)
 - [Apparmor](system_hardering/apparmor.md)
 - [Seccomp](system_hardering/seccomp.md)
 - [Limit node access](system_hardering/limit_node_access.md)
 - [SSH hardering](system_hardering/ssh_hardering.md)
 - [Minimize external access to network](system_hardering/minimize_network_access.md)
 - [Restrict Kernel modules](system_hardering/restrict_kernel.md)
 - [Linux privilege escalation](system_hardering/linux_privileges.md)



## Minimize microservice Vulnerabilities

 - [Security Contexts](minimize_microservice_vulnerabilities/security_context.md)
 - [Adminssions Controllers](minimize_microservice_vulnerabilities/admission_controllers.md)
 - [Pod security policies](minimize_microservice_vulnerabilities/pod_sec_policies.md)
 - [Open Policy Agent OPA](minimize_microservice_vulnerabilities/opa.md)
 - [gVisor](minimize_microservice_vulnerabilities/gvisor.md)
 - [kata Containers](minimize_microservice_vulnerabilities/kata_containers.md)
 - [Control Plane isolation](minimize_microservice_vulnerabilities/controlplane_isolation.md)
 - [Data plane Isolation](minimize_microservice_vulnerabilities/dataplane_isolation.md)
 - [Pod-to-pod encryption](minimize_microservice_vulnerabilities/pod-to-pod-encryption.md)
 - [Cilium](minimize_microservice_vulnerabilities/cilium.md)
 - [QoS](minimize_microservice_vulnerabilities/qos.md)

## Supply chain security
 
 - [Supply chain security](supply_chain_security/supply_chain_security.md)
 - [SBOM](supply_chain_security/sbom.md)
 - [Trivy](supply_chain_security/trivy.md)
 - [Image security](supply_chain_security/image_security.md)
 - [Kubesec scan](supply_chain_security/kubesec.md)
   


## Monitoring, Logging and Runtime security

  - [Falco](monitoring_logging/falco.md)
  - [Mutable and Immutable infrastructure](monitoring_logging/mutable_immutable_infra.md)
  - [Sysdig](monitoring_logging/sysdig.md)
  - [Audit log](monitoring_logging/audit_log.md)


## Exam tips

 - [External monitor for CKS/CKS/CKAD exams](https://www.reddit.com/r/kubernetes/comments/w5h1u6/my_cka_exam_is_tomorrow_can_i_use_external/) 

## Official Documentation

- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes Security Documentation](https://kubernetes.io/docs/concepts/security/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/best-practices/)
- [Kubernetes Security Checklist](https://kubernetes.io/docs/concepts/security/security-checklist/)

## Community-Driven Study Guides

- [CKS Preparation Guide (GitHub)](https://github.com/leandrocostam/cks-preparation-guide)
- [CKS Exam Study Guide by DevOpsCube](https://devopscube.com/cks-exam-guide-tips/)

## Video Resources

- [PASS your Certified Kubernetes Security Specialist Exam](https://www.youtube.com/watch?v=_l232KiJHNA)
- [Kubernetes CKS Full Course Theory + Practice + Browser Scenarios](https://www.youtube.com/watch?v=d9xfB5qaOfg)
- [How to pass the CKS exam](https://www.youtube.com/watch?v=2CnZarRBBBE)

