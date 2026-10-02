## Chapter 2: RBAC Least Privilege & Service Accounts

### Object Scope Matrix
| Object | Scope | Can Reference |
|---|---|---|
| Role | Single namespace | N/A (defines rules) |
| ClusterRole | Cluster-wide (or namespace via RoleBinding) | N/A (defines rules) |
| RoleBinding | Single namespace | Role OR ClusterRole |
| ClusterRoleBinding | Cluster-wide | ClusterRole ONLY |

**Key pattern:** ClusterRole + RoleBinding = reusable role definition applied only in one namespace. This is NOT the same as cluster-wide access.

### Core Commands
```bash
# Create Role
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev

# Create RoleBinding to a ServiceAccount
kubectl create rolebinding pod-reader-binding \
  --role=pod-reader --serviceaccount=dev:app-sa -n dev

# Create ClusterRole
kubectl create clusterrole node-reader --verb=get,list --resource=nodes

# Create ClusterRoleBinding
kubectl create clusterrolebinding node-reader-binding \
  --clusterrole=node-reader --serviceaccount=dev:app-sa

# Test permissions (MUST use full SA identity)
kubectl auth can-i delete secrets \
  --as=system:serviceaccount:dev:app-sa -n dev

# List all effective permissions
kubectl auth can-i --list --as=system:serviceaccount:dev:app-sa -n dev
```

### automountServiceAccountToken
```yaml
# On ServiceAccount (applies to all its pods)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
automountServiceAccountToken: false
---
# On Pod (overrides ServiceAccount setting)
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: app-sa
  automountServiceAccountToken: false
```
- Default value if unset: `true`
- Pod-level setting takes precedence over ServiceAccount-level.
- If a pod doesn't call the API server, always disable this.

### Avoiding Wildcards — Bad vs Good
```yaml
# BAD — overly broad
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

# GOOD — explicit least privilege
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### Aggregated ClusterRoles
```yaml
kind: ClusterRole
metadata:
  name: custom-monitoring
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: []  # auto-populated by controller
```

### Exam Gotchas
1. `--as=app-sa` (missing `system:serviceaccount:ns:` prefix) impersonates a **User**, not the ServiceAccount — always wrong on the exam.
2. ClusterRoleBinding can NEVER reference a Role — only ClusterRole.
3. Wildcard in **any** of apiGroups/resources/verbs is a hardening finding, even if the other two fields look scoped.
4. `automountServiceAccountToken` is not a container-level or metadata-level field — Pod spec or ServiceAccount object only.
