## Chapter 5 Cheat Sheet — PSA & Policy Engines

### Pod Security Admission (PSA) Levels
| Level | Behavior |
|---|---|
| privileged | Unrestricted — allows everything (host namespaces, privileged containers, hostPath) |
| baseline | Blocks known privilege escalations (no privileged containers, no host namespaces, restricted hostPath/volume types) |
| restricted | Hardened: requires runAsNonRoot, allowPrivilegeEscalation=false, seccompProfile set, all capabilities dropped |

### Namespace Labels (PSA)
```
kubectl label ns prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```
| Mode | Effect |
|---|---|
| enforce | Blocks non-compliant pods |
| audit | Admits pod, logs violation to audit log |
| warn | Admits pod, returns warning to client (kubectl) |

Optional version pin: `pod-security.kubernetes.io/enforce-version=latest` (or `v1.30` etc).

### PSP Migration
- PSP deprecated: v1.21
- PSP **removed**: v1.25 (no API, no controller)
- Migration path: PSA labels for baseline coverage + Kyverno/Gatekeeper for anything PSA doesn't express (e.g. custom label requirements, registry allow-lists)

### Kyverno ClusterPolicy
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-privileged
spec:
  validationFailureAction: Audit   # or Enforce
  rules:
  - name: check-privileged
    match:
      resources:
        kinds: [Pod]
    validate:
      message: "Privileged containers are not allowed"
      pattern:
        spec:
          containers:
          - =(securityContext):
              =(privileged): "false"
```
| Rule type | Purpose |
|---|---|
| validate | Allow/deny based on pattern match |
| mutate | Rewrite incoming resource (e.g. inject defaults) |
| generate | Auto-create companion resources (e.g. NetworkPolicy) |
| verifyImages | Check image signatures/provenance |

### OPA Gatekeeper Pattern
1. **ConstraintTemplate** — defines Rego + CRD schema, generates new CRD kind
2. **Constraint** — instance of that CRD, sets `match` + `parameters`
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8srequiredlabels
      violation[{"msg": msg}] { ... }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds: [{apiGroups: [""], kinds: ["Namespace"]}]
  parameters:
    labels: ["team"]
```

### Testing / Dry-Run
- Kyverno: `validationFailureAction: Audit` → check `PolicyReport`/`ClusterPolicyReport`
- Gatekeeper: `enforcementAction: dryrun` on Constraint → check `status.violations`
- Always test in a non-critical namespace before cluster-wide Enforce/deny rollout

### Exam Traps
- Confusing `audit` (logs only) with `enforce` (blocks) — audit does **not** stop the pod
- Assuming PSP still exists post-v1.25 — it's fully removed, not just deprecated
- Mixing up ConstraintTemplate (logic) vs Constraint (instance/parameters)
- Forgetting Gatekeeper has no native `generate` rule type — that's Kyverno-only
