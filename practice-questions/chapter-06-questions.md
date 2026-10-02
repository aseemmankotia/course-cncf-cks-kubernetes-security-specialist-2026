## Chapter 6: Microservice Security II: mTLS & Secrets Management — Practice Questions

**Q1.** A security audit finds that Kubernetes Secrets are stored in etcd as plaintext (base64 only). Which resource must be created and referenced by kube-apiserver to encrypt Secret data at rest?

A) A PodSecurityPolicy restricting hostPath volumes
B) An EncryptionConfiguration file referenced via the --encryption-provider-config flag
C) A NetworkPolicy denying egress from etcd pods
D) A ResourceQuota limiting Secret object count

<details>
<summary>Answer</summary>

**Correct: B**

EncryptionConfiguration, passed to kube-apiserver via --encryption-provider-config, defines providers (identity, aescbc, secretbox, kms) that encrypt Secret data before it is written to etcd.

**Why the others are wrong:**
- PodSecurityPolicy controls pod-level security context, not etcd storage encryption
- NetworkPolicy controls network traffic, not data-at-rest encryption
- ResourceQuota limits object counts/resources, unrelated to encryption

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A developer injects DB_PASSWORD into a container using env: valueFrom: secretKeyRef. A security engineer flags this as risky. Why is mounting the Secret as a volume preferred over env vars?

A) Environment variables cannot reference a secretKeyRef at all
B) Environment variable values are readable via kubectl describe pod, /proc/<pid>/environ, and commonly leak into logs and crash dumps
C) Environment variables always require cluster-admin RBAC to create
D) Environment variables are automatically shared with all namespaces

<details>
<summary>Answer</summary>

**Correct: B**

Env vars are exposed via process inspection, kubectl describe, and application logging/crash reporting, making them easier to accidentally leak than a mounted tmpfs volume file.

**Why the others are wrong:**
- secretKeyRef in env is a supported and common pattern, just insecure
- RBAC requirements are the same regardless of consumption method
- Secrets are namespace-scoped and never auto-shared across namespaces

</details>

---

**Q3.** A cluster uses Cilium as its CNI. The security team requires all pod-to-pod traffic to be encrypted and mutually authenticated without adding a sidecar proxy to every pod. Which approach meets this requirement?

A) Deploy an Istio sidecar with PeerAuthentication set to STRICT in every namespace
B) Add a 'tls: true' annotation to each Service object
C) Terminate TLS only at the Ingress controller and rely on cluster-internal trust
D) Enable Cilium's transparent encryption (WireGuard or IPsec) with identity-aware CiliumNetworkPolicy enforcement

<details>
<summary>Answer</summary>

**Correct: D**

Cilium can provide transparent node/pod traffic encryption via WireGuard or IPsec combined with identity-based network policy, achieving pod-to-pod protection at the CNI layer without sidecars.

**Why the others are wrong:**
- Istio sidecar mTLS is a valid alternative but explicitly requires sidecars, contradicting the requirement
- There is no such native Service annotation that provides pod-to-pod mTLS
- Ingress-only TLS termination leaves east-west (pod-to-pod) traffic unencrypted

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** Only the 'payments' ServiceAccount in the 'prod' namespace should be able to read the Secret named 'db-creds'. Which RBAC configuration enforces this most tightly?

A) A Role in 'prod' with resourceNames: ["db-creds"] and verbs: ["get"], bound to the ServiceAccount via a RoleBinding
B) A ClusterRole with verbs: ["get","list","watch"] on secrets bound via ClusterRoleBinding
C) A NetworkPolicy restricting traffic to the Secret's etcd key
D) A PodSecurityPolicy field named allowedSecrets listing db-creds

<details>
<summary>Answer</summary>

**Correct: A**

A namespaced Role with resourceNames restricts get access to only the named Secret, and a RoleBinding scopes this to the specific ServiceAccount and namespace — least privilege.

**Why the others are wrong:**
- A ClusterRole/ClusterRoleBinding without resourceNames grants access to ALL secrets cluster-wide, violating least privilege
- NetworkPolicy operates on network traffic, not the Kubernetes API/etcd objects
- PodSecurityPolicy has no field for restricting Secret access and is also deprecated/removed since v1.25

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** An organization wants to avoid storing secrets in Kubernetes etcd entirely, fetching them dynamically from HashiCorp Vault and keeping rotations in sync at the pod level. Which mechanism fits this requirement?

A) Native Kubernetes Secret objects with base64 encoding
B) A ConfigMap containing the Vault server URL
C) EncryptionConfiguration using the identity provider
D) Secrets Store CSI Driver with the Vault provider

<details>
<summary>Answer</summary>

**Correct: D**

The Secrets Store CSI Driver mounts secrets from external stores like Vault directly as a volume at pod startup, bypassing native Kubernetes Secret storage in etcd.

**Why the others are wrong:**
- Native Secrets still persist data (base64-encoded) in etcd, which is what the requirement avoids
- A ConfigMap only stores a URL — it does not fetch or inject the secret value
- identity provider disables encryption entirely and is unrelated to external secret fetching

</details>

---

**Q6.** Which EncryptionConfiguration provider is recommended for production because it integrates with an external Key Management Service instead of storing the encryption key in a local file on the control-plane node?

A) aescbc
B) secretbox
C) kms (KMSv2)
D) identity

<details>
<summary>Answer</summary>

**Correct: C**

The kms provider performs envelope encryption using an external KMS plugin, so the data encryption key is never persisted in a local config file on disk.

**Why the others are wrong:**
- aescbc stores its key directly in the EncryptionConfiguration file on the control-plane node
- secretbox also stores its key locally in the config file, same limitation as aescbc
- identity performs no encryption at all — it stores data in plaintext

</details>

---

**Q7.** You enabled EncryptionConfiguration with the aescbc provider and restarted kube-apiserver. Secrets created before this change are still stored unencrypted in etcd. Which command forces re-encryption of all existing Secrets?

A) kubectl apply -f encryptionconfig.yaml
B) kubectl get secrets --all-namespaces -o json | kubectl replace -f -
C) kubectl rollout restart deployment -n kube-system
D) etcdctl defrag

<details>
<summary>Answer</summary>

**Correct: B**

Reading and replacing every Secret triggers a write through the apiserver, which applies the new EncryptionConfiguration provider to that object as it is persisted to etcd.

**Why the others are wrong:**
- Applying the config file only affects the apiserver's active provider list, not existing stored objects
- Restarting deployments in kube-system does not touch Secret objects in etcd
- etcdctl defrag reclaims disk space; it does not re-encrypt data

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the key operational difference between mounting a Secret as a volume versus injecting it as an environment variable when the Secret's value is updated?

A) Environment variables refresh automatically, while volume mounts require a pod restart
B) Volume-mounted Secrets are automatically refreshed by kubelet within its sync period, while environment variable values remain stale until the pod is restarted
C) Both refresh automatically at the same rate with no restart needed
D) Neither refreshes without recreating the entire node

<details>
<summary>Answer</summary>

**Correct: B**

kubelet periodically syncs updated Secret data into mounted volumes (subject to kubelet sync period/cache), whereas env vars are only populated once at container start and never change without a restart.

**Why the others are wrong:**
- This reverses the actual behavior — env vars are the ones that stay stale
- Volume updates are not instantaneous nor identical in behavior to env vars
- Node recreation is never required for Secret propagation

</details>

---

