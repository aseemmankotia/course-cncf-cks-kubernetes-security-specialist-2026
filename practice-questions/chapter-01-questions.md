## Chapter 1: Cluster Setup: NetworkPolicy, CIS Benchmarks & TLS Ingress — Practice Questions

**Q1.** A namespace 'prod' has a default-deny-all NetworkPolicy applied (podSelector: {} with Ingress and Egress policyTypes). Pods in this namespace now fail to resolve DNS names. What is the MOST LIKELY cause and fix?

A) CoreDNS is crashing; restart the coredns deployment in kube-system
B) NetworkPolicy does not affect DNS traffic; the issue is unrelated to the policy
C) The default-deny policy blocks egress to kube-dns on port 53; add an explicit egress rule allowing UDP/TCP 53 to the kube-system namespace
D) Pods need a ServiceAccount token to resolve DNS; regenerate the token

<details>
<summary>Answer</summary>

**Correct: C**

A default-deny egress policy blocks ALL outbound traffic including DNS lookups to CoreDNS in kube-system. You must add an explicit egress rule permitting UDP/TCP port 53, typically scoped with a namespaceSelector matching kube-system.

**Why the others are wrong:**
- CoreDNS crashing is unrelated — the symptom appears immediately after applying the deny policy, a strong exam signal it's the policy
- NetworkPolicy absolutely affects DNS traffic since it's just another egress destination on port 53
- ServiceAccount tokens have nothing to do with DNS resolution

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** You run `kube-bench run --targets=master` and see: '[FAIL] 1.2.16 Ensure that the admission control plugin AlwaysPullImages is set (Automated)'. Which remediation is correct?

A) Run `kubectl edit deployment kube-apiserver -n kube-system` and add the flag
B) Restart the kubelet service; admission plugins are configured there
C) Edit /etc/kubernetes/manifests/kube-apiserver.yaml and add AlwaysPullImages to the --enable-admission-plugins flag, then let kubelet restart the static pod
D) Add the flag to /etc/kubernetes/admission-control.yaml, a separate config file

<details>
<summary>Answer</summary>

**Correct: C**

The API server runs as a static pod defined in the manifest file. Editing --enable-admission-plugins there and saving triggers kubelet to detect the change and restart the pod automatically — no kubectl needed.

**Why the others are wrong:**
- kube-apiserver is a static pod, not a Deployment resource — kubectl edit deployment will fail or find nothing
- Restarting kubelet does not itself change API server flags; the manifest must be edited
- There is no separate admission-control.yaml file in standard kubeadm clusters

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A security review requires that pods cannot reach the cloud provider's instance metadata service (169.254.169.254) to prevent credential theft via SSRF. Which control BEST enforces this at the cluster level?

A) Apply a NetworkPolicy with an egress rule that denies traffic to 169.254.169.254/32 for all pods in the namespace
B) Disable the pod's ServiceAccount token automount
C) Set a PodSecurityPolicy that disallows hostNetwork
D) Enable RuntimeDefault seccomp profile on all pods

<details>
<summary>Answer</summary>

**Correct: A**

A NetworkPolicy with an egress deny rule targeting the metadata IP range blocks pods from ever reaching the metadata endpoint, directly closing the SSRF-to-cloud-credentials path.

**Why the others are wrong:**
- Disabling automount protects the Kubernetes API token, not the cloud metadata service, which is a separate endpoint
- hostNetwork restriction prevents a different attack vector (host network access) but a pod on the pod network can still reach 169.254.169.254 via the node route
- seccomp restricts syscalls, not network destinations

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** You need to expose an internal web application externally over HTTPS, terminating TLS at the cluster edge rather than in the application pod. Which Kubernetes object should hold the TLS certificate and key?

A) A Secret of type kubernetes.io/tls, referenced in the Ingress spec.tls.secretName field
B) A ConfigMap referenced by the Deployment's volumeMounts
C) An environment variable injected into the pod spec
D) A Service of type LoadBalancer with an annotation containing the raw certificate

<details>
<summary>Answer</summary>

**Correct: A**

TLS termination at Ingress requires a Secret of type kubernetes.io/tls containing keys tls.crt and tls.key, referenced by name under spec.tls[].secretName in the Ingress resource.

**Why the others are wrong:**
- ConfigMaps are for non-sensitive data and are not consumed by the Ingress controller for TLS
- Environment variables would only apply if TLS terminated in-app, which contradicts the requirement
- Service annotations do not carry certificate material in standard Kubernetes; that's a cloud-LB-specific pattern, not the Ingress TLS mechanism

</details>

---

**Q5.** Before installing a new kubeadm binary on a node, you must verify its integrity per CIS hardening guidance. Which command correctly performs this check?

A) kubeadm version --verify
B) kubectl verify-binary kubeadm
C) sha256sum kubeadm and compare the output against the published checksum file (e.g. kubeadm.sha256) from the official release
D) gpg --recv-keys kubeadm && kubeadm --self-check

<details>
<summary>Answer</summary>

**Correct: C**

The CIS/CKS-recommended process is to download the published checksum alongside the binary and run `sha256sum kubeadm` then diff it against the checksum file contents to confirm the binary was not tampered with in transit.

**Why the others are wrong:**
- No such flag exists on kubeadm for integrity verification
- kubectl has no binary-verification subcommand
- gpg --recv-keys is unrelated syntax and --self-check is not a real kubeadm flag

</details>

---

**Q6.** Which kube-bench command targets etcd-specific CIS checks only, on a node where etcd runs as a static pod?

A) kube-bench run --targets=master,node,etcd,policies
B) kube-bench check --component=etcd
C) kube-bench scan --etcd-only
D) kube-bench run --targets=etcd

<details>
<summary>Answer</summary>

**Correct: D**

`kube-bench run --targets=etcd` scopes the scan to the etcd CIS section only (config file permissions, --client-cert-auth, --peer-client-cert-auth, etc).

**Why the others are wrong:**
- This runs ALL sections, not etcd-only — it would still work but does not answer 'etcd checks only'
- `check` is not a valid kube-bench subcommand; the subcommand is `run`
- `--etcd-only` is not a real flag; kube-bench uses --targets

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Per the CIS Kubernetes Benchmark, what is the recommended file permission mode for the etcd data directory?

A) 700
B) 755
C) 644
D) 600

<details>
<summary>Answer</summary>

**Correct: A**

CIS recommends etcd data directory permissions of 700, owned by etcd:etcd, so only the etcd process user can read/write cluster secrets and data on disk.

**Why the others are wrong:**
- 755 allows group/other read+execute, exposing directory listing to other local users
- 644 is a file mode (no execute bit) and far too permissive for a directory holding cluster secrets
- 600 is close but omits the execute bit needed to traverse the directory; CIS specifies 700

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** In a namespace with NO NetworkPolicy objects applied at all, what is the default traffic behavior between pods?

A) All ingress and egress traffic is denied by default until a policy is created
B) Only traffic within the same namespace is allowed; cross-namespace traffic is denied
C) Only DNS and API server traffic is allowed by default
D) All ingress and egress traffic is allowed; Kubernetes has no default-deny behavior without an explicit policy

<details>
<summary>Answer</summary>

**Correct: D**

Kubernetes networking is flat and open by default. Without any NetworkPolicy resource selecting a pod, all ingress and egress traffic to/from that pod is permitted (assuming the CNI plugin supports NetworkPolicy at all).

**Why the others are wrong:**
- Default-deny only exists once you explicitly create a policy with podSelector: {} and no allow rules — it is never the out-of-the-box default
- Cross-namespace traffic is allowed by default too; namespace isolation must be explicitly enforced with policies
- There is no implicit allowlist for DNS/API server traffic; those work simply because nothing blocks them by default

</details>

---

