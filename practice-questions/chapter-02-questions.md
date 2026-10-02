## Chapter 2: Cluster Hardening I: RBAC Least Privilege & Service Accounts — Practice Questions

**Q1.** A microservice pod in namespace 'billing' must only be able to list and get Pods within its own namespace. Following least-privilege principles, which combination of RBAC objects should you create?

A) A Role and a RoleBinding, both scoped to the 'billing' namespace
B) A ClusterRole and a ClusterRoleBinding
C) A ClusterRole scoped with a namespace field, bound with a RoleBinding
D) A Role bound with a ClusterRoleBinding

<details>
<summary>Answer</summary>

**Correct: A**

Role and RoleBinding are namespace-scoped objects; using both keeps permissions confined to 'billing' with zero cluster-wide exposure — the textbook least-privilege pattern.

**Why the others are wrong:**
- ClusterRole+ClusterRoleBinding grants the permission cluster-wide across every namespace, violating least privilege.
- ClusterRole objects have no 'namespace' field — RBAC scoping never works that way; ClusterRoles are always cluster-scoped resources.
- A ClusterRoleBinding always grants cluster-wide access regardless of the referenced Role/ClusterRole kind — binding a Role to a ClusterRoleBinding is actually invalid; RoleBindings can reference ClusterRoles but ClusterRoleBindings cannot reference Roles at all.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** During an audit you find that a compromised pod's shell was used to curl the Kubernetes API server using the pod's mounted service account token, and successfully list Secrets. The pod does not need API access at all. Which single change best prevents this class of attack going forward?

A) Add a NetworkPolicy blocking egress to the pod
B) Set automountServiceAccountToken: false on the Pod spec or its ServiceAccount
C) Rotate the ServiceAccount token every 24 hours
D) Bind a ClusterRole with only 'get' verb instead of 'list'

<details>
<summary>Answer</summary>

**Correct: B**

If the pod never calls the API server, the token should never be mounted in the first place — automountServiceAccountToken: false removes the attack surface entirely.

**Why the others are wrong:**
- A NetworkPolicy on egress doesn't stop access to the in-cluster API server ClusterIP, which is typically still reachable, and doesn't address the root cause of an unneeded mounted token.
- Rotating tokens reduces the exposure window but the attacker can still use a valid token for API calls during its lifetime — it doesn't prevent the access.
- Restricting verbs still allows the attacker to read Secrets via 'get'; the pod shouldn't have any binding to Secrets in the first place, and the mounted token is the real exposure vector.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** Before deploying a new manifest, you want to confirm whether ServiceAccount 'app-sa' in namespace 'dev' can delete Secrets in that namespace, without actually performing the delete. Which command achieves this?

A) kubectl get rolebindings -n dev -o yaml
B) kubectl describe serviceaccount app-sa -n dev
C) kubectl auth can-i delete secrets --as=system:serviceaccount:dev:app-sa -n dev
D) kubectl auth can-i delete secrets --as=app-sa -n dev

<details>
<summary>Answer</summary>

**Correct: C**

kubectl auth can-i performs a dry-run SelfSubjectAccessReview/SubjectAccessReview check; impersonating a ServiceAccount requires the full qualified name system:serviceaccount:<namespace>:<name> passed via --as.

**Why the others are wrong:**
- Listing RoleBindings only shows binding objects — it doesn't tell you the effective resolved permission and requires manual cross-referencing of Roles.
- describe serviceaccount shows secrets/tokens associated with the SA, not its RBAC permissions.
- --as=app-sa without the system:serviceaccount:<ns>: prefix is interpreted as impersonating a regular User named 'app-sa', not the actual ServiceAccount identity, producing an incorrect result.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A team needs a user to be able to create and list Pods across ALL namespaces in the cluster. Which RBAC object pairing correctly satisfies this requirement?

A) Role + ClusterRoleBinding
B) ClusterRole + RoleBinding
C) ClusterRole + ClusterRoleBinding
D) Role + RoleBinding

<details>
<summary>Answer</summary>

**Correct: C**

Only a ClusterRoleBinding grants permissions cluster-wide across every namespace, and it must reference a ClusterRole (or Role, but a Role's scope wouldn't matter since ClusterRoleBinding forces cluster scope) — the correct textbook pairing for cluster-wide access is ClusterRole+ClusterRoleBinding.

**Why the others are wrong:**
- ClusterRoleBindings cannot reference a Role object — only ClusterRole is a valid roleRef kind for a ClusterRoleBinding.
- A RoleBinding, even one referencing a ClusterRole, only grants that ClusterRole's permissions within the RoleBinding's own namespace — not all namespaces.
- Role+RoleBinding is entirely namespace-scoped and cannot grant cluster-wide access no matter how it's bound.

</details>

---

**Q5.** Which kubectl command correctly creates a namespace-scoped Role named 'pod-reader' in namespace 'prod' with only get, list, and watch permissions on pods (no wildcards)?

A) kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
B) kubectl create rolebinding pod-reader --verb=get,list,watch --resource=pods -n prod
C) kubectl create role pod-reader --verb=get,list,watch --resource=pods -n prod
D) kubectl apply -f pod-reader.yaml --verb=get,list,watch

<details>
<summary>Answer</summary>

**Correct: C**

kubectl create role generates a namespace-scoped Role object with explicit --verb and --resource flags, matching the exact requirement.

**Why the others are wrong:**
- create clusterrole produces a cluster-scoped object, not the requested namespace-scoped Role, and has no -n flag semantics for scoping.
- create rolebinding builds a binding object, not a Role — it doesn't accept --verb/--resource flags for defining permissions.
- kubectl apply does not accept --verb or --resource flags at all; those flags only work with the imperative 'create role/clusterrole' commands.

</details>

---

**Q6.** Which manifest fragment correctly disables the automatic mounting of a ServiceAccount token for a Pod?

A) spec:
  containers:
  - name: app
    automountServiceAccountToken: false
B) spec:
  automountServiceAccountToken: false
C) metadata:
  automountServiceAccountToken: false
D) spec:
  serviceAccount:
    automount: false

<details>
<summary>Answer</summary>

**Correct: B**

automountServiceAccountToken is a valid field directly under PodSpec (spec) — and also valid under the ServiceAccount object's top level — controlling whether the token volume is auto-mounted.

**Why the others are wrong:**
- automountServiceAccountToken is not a valid field on a container spec — the field only exists at the Pod or ServiceAccount object level.
- It is not a metadata field; metadata only holds labels, annotations, name, namespace, etc.
- There is no 'serviceAccount.automount' nested field in the PodSpec schema — this is a fabricated structure.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which Role definition avoids wildcard verbs and resources while still granting read-only access to Pods only?

A) rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list"]
B) rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]
C) rules:
- apiGroups: ["*"]
  resources: ["pods"]
  verbs: ["get", "list"]
D) rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

<details>
<summary>Answer</summary>

**Correct: D**

Explicit resource ('pods') and explicit verbs (get, list, watch) with no wildcards anywhere is the least-privilege, exam-correct pattern for read-only pod access.

**Why the others are wrong:**
- resources: ["*"] wildcards ALL resource types in the core API group, far exceeding a pods-only requirement.
- verbs: ["*"] wildcards ALL verbs including delete, create, and patch, violating read-only intent.
- apiGroups: ["*"] wildcards every API group in the cluster, an unnecessary and dangerous over-grant even though resources/verbs look scoped.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is an Aggregated ClusterRole?

A) A Role that combines permissions across multiple namespaces into one object
B) A ClusterRole whose permissions are automatically composed from other ClusterRoles matched via label selectors in an aggregationRule
C) A ClusterRoleBinding that attaches multiple subjects to one ClusterRole
D) A deprecated rbac.authorization.k8s.io/v1beta1 construct no longer supported

<details>
<summary>Answer</summary>

**Correct: B**

Aggregated ClusterRoles use an aggregationRule with clusterRoleSelectors; the control plane's ClusterRoleAggregation controller automatically unions matching ClusterRoles' rules into the aggregate — this is how built-in roles like admin/edit/view are extended by add-ons.

**Why the others are wrong:**
- Roles cannot span multiple namespaces — they are always scoped to a single namespace, so this describes a nonexistent object.
- That describes a normal ClusterRoleBinding with multiple subjects, unrelated to the aggregation mechanism.
- Aggregation is a fully supported, current rbac.authorization.k8s.io/v1 feature, not a deprecated beta construct.

</details>

---

