# Open Policy Agent (OPA)

## What is Open Policy Agent (OPA)?
Open Policy Agent (OPA) is a general-purpose policy engine that enables fine-grained, declarative access control across the Kubernetes ecosystem.

## Why Use OPA in Kubernetes?
- Enforce security policies dynamically.
- Decouple policy decisions from application code.
- Centralized policy management with **Rego** language.

## OPA + Kubernetes Integration
OPA is commonly deployed with **Gatekeeper**, which integrates with Kubernetes Admission Controllers.

## Installing OPA Gatekeeper
```sh
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```

## Creating a ConstraintTemplate (Defining a Policy)
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: requiredlabels
spec:
  crd:
    spec:
      names:
        kind: RequiredLabels
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package requiredlabels
        violation[{"msg": msg}] {
          not input.review.object.metadata.labels["team"]
          msg := "Missing required label: team"
        }
```

## Creating a Constraint (Enforcing the Policy)
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequiredLabels
metadata:
  name: enforce-team-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters: {}
```

## Testing the Policy
```sh
kubectl apply -f pod-without-label.yaml
```
Expected Output:
```
Error from server: admission webhook "validation.gatekeeper.sh" denied the request: 
Missing required label: team
```

## Debugging OPA Policies
- Check Gatekeeper logs:
  ```sh
  kubectl logs -n gatekeeper-system -l control-plane=controller-manager
  ```
- List active constraints:
  ```sh
  kubectl get constraints
  ```
- Dry-run policies using `opa eval`:
  ```sh
  opa eval --data policy.rego --input input.json "data.requiredlabels.violation"
  ```

## Best Practices for OPA in Kubernetes
- Use **Gatekeeper** for admission control.
- Write clear and modular **Rego** policies.
- Regularly audit policies using `kubectl get constraints`.
- Monitor Gatekeeper logs for policy enforcement issues.
- Set `failurePolicy: Ignore|Fail` carefully to control webhook behavior.

---
**Reference**: [OPA Gatekeeper Docs](https://open-policy-agent.github.io/gatekeeper/website/docs/)
