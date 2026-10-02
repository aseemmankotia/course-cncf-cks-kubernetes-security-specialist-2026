## Chapter 3: Cluster Hardening II: Securing the API Server & Kubelet — Practice Questions

**Q1.** You are auditing a cluster and find port 10255 responding to unauthenticated GET requests on every worker node. Which action BEST remediates this?

A) Set --anonymous-auth=false on the kube-apiserver
B) Set --read-only-port=0 in the kubelet config and restart kubelet
C) Create a NetworkPolicy blocking traffic to port 10255
D) Rotate the kubelet client certificate

<details>
<summary>Answer</summary>

**Correct: B**

Port 10255 is a separate, always-unauthenticated read-only kubelet API. It bypasses auth flags entirely; the only fix is to disable it with --read-only-port=0.

**Why the others are wrong:**
- --anonymous-auth belongs to the kubelet (or apiserver for a different purpose) and does not affect the read-only port, which has no auth checks at all.
- NetworkPolicy only governs pod-to-pod traffic via CNI, not host-level kubelet ports.
- Rotating a client cert has no effect on an unauthenticated port that ignores certs.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** Which combination of kubelet flags enforces authenticated AND authorized access to the kubelet API?

A) --anonymous-auth=false, --authorization-mode=Webhook
B) --anonymous-auth=true, --authorization-mode=AlwaysAllow
C) --anonymous-auth=false, --authorization-mode=AlwaysAllow
D) --read-only-port=10255, --authorization-mode=Webhook

<details>
<summary>Answer</summary>

**Correct: A**

Disabling anonymous auth forces all requests to authenticate, and Webhook mode delegates the authorization decision to the API server via SubjectAccessReview.

**Why the others are wrong:**
- AlwaysAllow bypasses authorization entirely regardless of authentication — a classic exam trap.
- Even with anonymous-auth disabled, AlwaysAllow still permits any authenticated caller to do anything.
- Leaving --read-only-port=10255 open reopens the unauthenticated endpoint, contradicting the hardening goal.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** What is the correct --authorization-mode value for kube-apiserver so that Node authorization and RBAC are both enforced?

A) --authorization-mode=RBAC,Node
B) --authorization-mode=Webhook,RBAC
C) --authorization-mode=ABAC,Node
D) --authorization-mode=Node,RBAC

<details>
<summary>Answer</summary>

**Correct: D**

Node,RBAC is the documented, standard hardened configuration: the Node authorizer specifically evaluates kubelet requests, paired with RBAC for general subjects.

**Why the others are wrong:**
- Reversing the order is non-standard on the exam and not the documented pattern for kubeadm-hardened clusters.
- Webhook mode delegates to an external service and is unrelated to enforcing Node-level restriction.
- ABAC is a legacy, deprecated mode flagged as an insecure choice on the exam.

</details>

---

**Q4.** What is the primary purpose of the NodeRestriction admission controller?

A) Prevents a kubelet from modifying Node and Pod objects other than its own
B) Encrypts etcd data at rest
C) Restricts which container images can run on a node
D) Enforces NetworkPolicy rules at the node level

<details>
<summary>Answer</summary>

**Correct: A**

NodeRestriction limits kubelets to modifying only their own Node object and Pods bound to them, and blocks setting node-restriction.kubernetes.io/ labels.

**Why the others are wrong:**
- Encryption at rest is handled by EncryptionConfiguration, not NodeRestriction.
- Image restriction is enforced by ImagePolicyWebhook or admission policies, not NodeRestriction.
- NetworkPolicy enforcement is a CNI plugin function, unrelated to this admission controller.

</details>

---

**Q5.** Your etcd cluster runs with --client-cert-auth=false, allowing any client to read Secrets directly from etcd. Which single change closes this hole while keeping the cluster operational?

A) Enable encryption at rest for the Secrets resource
B) Delete the etcd data directory and rebuild the cluster
C) Set --peer-client-cert-auth=false to simplify peer trust
D) Set --client-cert-auth=true and configure --trusted-ca-file

<details>
<summary>Answer</summary>

**Correct: D**

Requiring client cert auth forces every client (including the API server) to present a certificate signed by a trusted CA before etcd will serve requests.

**Why the others are wrong:**
- Encryption at rest protects data on disk but does not stop a live, unauthenticated client from querying the running etcd API.
- Rebuilding the cluster is destructive, causes data loss, and does not address the missing auth configuration.
- Disabling peer cert auth weakens security further and has nothing to do with client access control.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** Which etcd flag enables mutual TLS authentication between etcd cluster members (peers)?

A) --client-cert-auth=true
B) --peer-client-cert-auth=true
C) --cert-file and --key-file only
D) --insecure-transport=false

<details>
<summary>Answer</summary>

**Correct: B**

--peer-client-cert-auth=true, combined with --peer-cert-file/--peer-key-file/--peer-trusted-ca-file, enforces mTLS for inter-member (peer) traffic specifically.

**Why the others are wrong:**
- --client-cert-auth governs client-to-etcd (e.g., API server) authentication, not peer-to-peer traffic.
- Setting cert/key files alone configures the server certificate but does not require peers to present a valid certificate.
- --insecure-transport is not a real etcd flag; it's a distractor.

</details>

---

**Q7.** A kube-apiserver runs v1.31 and one worker node's kubelet is still on v1.27. What must the administrator do to comply with the Kubernetes version skew policy?

A) Nothing — v1.27 kubelet is within supported skew of a v1.31 API server
B) Upgrade the kubelet to at least v1.28 to stay within the supported n-3 minor version skew
C) Downgrade the API server to v1.27 to match the kubelet
D) Upgrade only the kubelet to v1.32

<details>
<summary>Answer</summary>

**Correct: B**

The kubelet-to-API-server skew policy allows the kubelet to be up to 3 minor versions older (n-3). v1.31 minus 3 is v1.28, so a v1.27 kubelet is 4 versions behind and unsupported — it must be upgraded to at least v1.28.

**Why the others are wrong:**
- A 4-minor-version gap exceeds the supported n-3 skew, making this configuration invalid.
- Downgrading the API server abandons required security patches and contradicts the 'keep versions current' best practice.
- A kubelet must never be newer than the API server, so jumping the kubelet to v1.32 violates the skew policy in the other direction.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which authorization mode should be avoided on a hardened kube-apiserver because it relies on a static policy file and is difficult to audit or update dynamically?

A) RBAC
B) Node
C) ABAC
D) Webhook

<details>
<summary>Answer</summary>

**Correct: C**

ABAC is a legacy mode using a manually edited policy file requiring an API server restart to change, offering poor auditability compared to RBAC.

**Why the others are wrong:**
- RBAC is the recommended, dynamically manageable standard for hardened clusters.
- Node authorization is required alongside RBAC to properly restrict kubelet requests.
- Webhook mode is a valid, supported option for delegating authorization to an external service — not deprecated.

</details>

---

