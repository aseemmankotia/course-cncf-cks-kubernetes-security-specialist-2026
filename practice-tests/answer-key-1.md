# Answer Key — Practice Test 1

**Q1. Correct: C**

Placing namespaceSelector and podSelector inside the SAME 'from' list item ANDs them together — traffic is only allowed if the source pod matches BOTH conditions (in analytics namespace AND labeled role=reporting). policyTypes: [Ingress] with an empty podSelector applies the policy to all pods in 'finance' and defaults to deny-all-ingress except what's explicitly allowed.

---

**Q2. Correct: B**

'--client-cert-auth=true' forces etcd to require and validate client certificates for client-to-server (API server-to-etcd) connections, which is exactly what CIS benchmark check 2.2 tests. Without it, etcd may accept unauthenticated client connections even if TLS certs are configured.

---

**Q3. Correct: C**

Setting 'anonymous.enabled: false' rejects unauthenticated requests to the kubelet API, and 'authorization.mode: Webhook' delegates authorization decisions to the API server (via SubjectAccessReview) instead of allowing every request. This combination satisfies both the CIS kubelet authentication and authorization checks.

---

**Q4. Correct: D**

Ingress TLS termination requires a Secret of type 'kubernetes.io/tls', which stores the certificate under the 'tls.crt' key and the private key under 'tls.key'. Ingress controllers specifically look for these standardized key names in a tls-typed secret referenced by secretName.

---

**Q5. Correct: B**

A NetworkPolicy egress rule that explicitly denies traffic to the 169.254.169.254 metadata IP directly blocks pods from querying the cloud instance metadata endpoint, preventing credential theft. This is the standard CKS remediation for the 'protect node metadata and endpoints' objective.

---

**Q6. Correct: D**

Kubernetes release artifacts are published alongside SHA-256 checksum files. Running 'sha256sum kubeadm' locally and comparing the output to the officially published value cryptographically confirms the binary is authentic and unmodified — this is the documented supply-chain verification step for platform binaries.

---

**Q7. Correct: D**

A NetworkPolicy with an egress deny rule targeting the metadata IP (169.254.169.254/32) blocks pods from reaching the instance metadata service at the network layer, which is the standard CKS-tested mitigation for metadata/endpoint protection, while still allowing other required egress traffic via explicit allow rules or exceptions for pods that legitimately need cloud API access (e.g., via IRSA/workload identity).

---

**Q8. Correct: D**

Setting automountServiceAccountToken: false on the ServiceAccount (or overriding it per-Pod) is the supported mechanism to stop the API server credentials and CA bundle from being auto-mounted into pods that don't need them, satisfying least privilege without breaking scheduling.

---

**Q9. Correct: A**

--anonymous-auth=false disables the system:anonymous user and system:unauthenticated group, so every request must pass an authentication method (client cert, token, etc.) before authorization is even evaluated.

---

**Q10. Correct: A**

The kubelet's authentication.anonymous.enabled field (set via the kubelet config file or --anonymous-auth=false flag) controls whether unauthenticated requests are treated as system:anonymous; setting it false forces authentication for every kubelet API call.

---

**Q11. Correct: D**

A namespaced Role defining get/list on pods, bound to the ServiceAccount with a RoleBinding in the build namespace, grants exactly the required permissions and nowhere else, which is the textbook least-privilege RBAC pattern the exam expects.

---

**Q12. Correct: C**

NodeRestriction limits a kubelet's write access via the system:node authorization group so it can only modify its own Node object and the Pod objects bound to it, preventing a compromised node from tampering with other nodes' status or scheduling other nodes' pods.

---

**Q13. Correct: A**

--client-cert-auth=true forces etcd to require and validate a client TLS certificate against the trusted CA (--trusted-ca-file) for every client connection, which is exactly the requirement of authenticating callers like the kube-apiserver before allowing access to stored cluster data.

---

**Q14. Correct: B**

Setting --anonymous-auth=false forces every request to the kubelet API to be authenticated, and --authorization-mode=Webhook delegates the authorization decision to the API server, which checks RBAC rules (via SubjectAccessReview) before allowing operations like exec or logs. This is the documented CIS Benchmark and CKS-recommended kubelet hardening configuration.

---

**Q15. Correct: B**

Since Kubernetes 1.30 (GA), AppArmor is configured natively via the container's securityContext.appArmorProfile field, with type Localhost and localhostProfile pointing to the profile name already loaded on the node. This replaced the older beta annotation approach.

---

**Q16. Correct: D**

type: Localhost tells the kubelet to load a custom profile, and localhostProfile gives the path relative to the kubelet's seccomp root directory (/var/lib/kubelet/seccomp/), so profiles/audit.json resolves to the file described in the scenario.

---

**Q17. Correct: C**

Minimizing the host OS footprint by using a stripped-down, purpose-built node OS and disabling/removing unnecessary kernel modules, packages, and services directly shrinks the number of exploitable binaries, drivers, and listening services on the node itself — this is the core System Hardening domain objective.

---

**Q18. Correct: C**

SSH is a host-level service, not a Kubernetes-managed resource, so restricting access to it requires host/OS-level or infrastructure-level firewalling (iptables, nftables, or a cloud provider security group) scoped to the bastion IP — this directly reduces external node access per the System Hardening domain.

---

**Q19. Correct: A**

AppArmor is a Linux Security Module providing mandatory access control based on file paths, capabilities, and network access rules; seccomp operates at the syscall filtering layer, allowing/denying specific syscalls a process can make. Both are applied per-container via securityContext and are distinct, complementary hardening layers on CKS.

---

**Q20. Correct: D**

The built-in Pod Security Admission (PSA) controller supports independent audit, warn, and enforce modes via namespace labels. Staging audit/warn before enforce lets teams find violations without breaking running workloads, and this requires no third-party controller.

---

**Q21. Correct: D**

gVisor implements a user-space kernel (Sentry) that intercepts application syscalls before they reach the host kernel, integrates via containerd's runtime shim v2, and is designed specifically for this isolation model.

---

**Q22. Correct: A**

Enforce mode causes the admission request to be denied when the validate rule pattern is not matched, correctly blocking non-conforming Pods at admission time.

---

**Q23. Correct: A**

Gatekeeper's policy engine (OPA) uses Rego expressed through ConstraintTemplates and Constraint CRDs, whereas Kyverno policies are authored as declarative Kubernetes YAML with validate/mutate/generate rules and no Rego required.

---

**Q24. Correct: A**

Encryption at rest for Secrets is configured via an EncryptionConfiguration file referenced by --encryption-provider-config on the API server; existing Secrets must then be re-written (e.g., via a bulk update) so etcd stores them encrypted rather than in base64 plaintext.

---

**Q25. Correct: C**

Cilium, as an eBPF-based CNI, can transparently encrypt node-to-node pod traffic using WireGuard or IPsec at the datapath level, requiring no application changes and no sidecar containers.

---

**Q26. Correct: A**

Kyverno policies are expressed as ClusterPolicy (or Policy for namespace scope) custom resources. A validate rule with a pattern block can require spec.containers[*].resources.limits to be set, and Kyverno's admission controller enforces it automatically.

---

**Q27. Correct: A**

gVisor implements a user-space kernel (the Sentry) that intercepts and re-implements Linux syscalls, providing isolation without a full VM. Its OCI-compatible runtime binary is 'runsc', which is the handler name typically used in a Kubernetes RuntimeClass.

---

**Q28. Correct: D**

Encryption at rest for Secrets is configured on the kube-apiserver via the `--encryption-provider-config` flag, pointing to a local EncryptionConfiguration file that defines providers such as aescbc with a base64-encoded key, applied to the secrets resource.

---

**Q29. Correct: B**

Distroless images remove the shell, package manager, and unnecessary OS utilities while still including only the minimal runtime dependencies needed to execute the application, directly reducing the attack surface without breaking functionality.

---

**Q30. Correct: A**

hadolint is a dedicated Dockerfile linter that parses Dockerfile instructions and flags best-practice violations such as missing USER directives, use of ADD, and root execution, before any image is built.

---

**Q31. Correct: D**

OPA Gatekeeper constraints (or native ValidatingAdmissionPolicy in current Kubernetes) can inspect the container image field on every Pod at admission time and reject the request if the image reference does not start with the approved registry prefix, providing a hard block cluster-wide.

---

**Q32. Correct: B**

trivy scans image layers for known CVEs, and using --severity CRITICAL with --exit-code 1 causes the pipeline command to return a non-zero exit code (failing the build) whenever a CRITICAL vulnerability is found.

---

**Q33. Correct: B**

Both SPDX and CycloneDX are widely adopted SBOM standards. SPDX grew out of the Linux Foundation's software license compliance work, while CycloneDX was created by OWASP with an emphasis on security and vulnerability management use cases; both are actively used by tools like syft, trivy, and cosign attestations.

---

**Q34. Correct: A**

cosign signs images and pushes the signature alongside the image in the registry. A policy engine like Kyverno's verifyImages rule or Sigstore's Policy Controller can then be configured at admission time with the corresponding public key to verify the signature and reject any Pod whose image is unsigned or fails verification.

---

**Q35. Correct: B**

An admission controller that validates the image field against an allowed-registry policy is the only mechanism that inspects and enforces the image source at object creation time, blocking non-conforming Pods before they are scheduled — this is the standard CKS pattern using OPA/Gatekeeper ConstraintTemplates or the built-in ImagePolicyWebhook.

---

**Q36. Correct: B**

kubesec.io is specifically designed to statically score Kubernetes manifests (Deployments, Pods, etc.) against a rule set covering securityContext settings, capabilities, resource limits, and volume types, returning a risk score with pass/fail rules — this is the exact CKS-tested static-analysis tool for manifests.

---

**Q37. Correct: B**

Signature verification must happen at admission time, before the Pod object is persisted and scheduled; Sigstore Policy Controller (or Connaisseur) is a validating admission webhook that checks image signatures against a configured trusted public key and rejects the request outright if verification fails, which is the CKS-tested pattern for enforcing signed-image supply chain policy.

---

**Q38. Correct: B**

Falco hooks into syscalls (via kernel module or eBPF probe) and ships a default rule, 'Terminal shell in container spawned', that fires when a shell process (bash, sh, etc.) is spawned inside a container, exactly the real-time behavioral detection required here.

---

**Q39. Correct: C**

The RequestResponse level logs the full event metadata plus the request and response bodies, which is required to capture exactly what Secret data was accessed or returned — necessary for forensic review of sensitive resource access.

---

**Q40. Correct: B**

With readOnlyRootFilesystem: true, the container's root filesystem (including /tmp unless a writable volume is explicitly mounted there) is mounted read-only at the OS level. Any write attempt fails immediately with EROFS / 'Read-only file system', enforcing container immutability at runtime without needing any external detection component.

---

**Q41. Correct: D**

Falco is the CNCF runtime security tool designed to detect anomalous behavior via syscall monitoring. Its eBPF probe driver avoids compiling a custom kernel module per node (important for managed/hardened nodes) while still providing kernel-level visibility with lower operational overhead than the legacy kernel module driver.

---

**Q42. Correct: C**

--audit-policy-file specifies the path to the audit policy defining what to log and at what level, and --audit-log-path specifies the file where JSON-formatted audit events are written. Both flags are required together for log-backend auditing; audit-log-format defaults to json.

---

**Q43. Correct: C**

Falco supports multiple drivers for capturing syscalls: the original kernel module, a userspace eBPF probe, and a modern eBPF driver. The eBPF-based drivers do not require compiling/loading a custom kernel module per node, making them well suited to managed clusters or hardened nodes where module loading is restricted.

---

**Q44. Correct: C**

Falco hooks into the kernel via eBPF/kernel module and inspects syscalls directly, so it detects a shell process being spawned no matter how it got there — kubectl exec, a reverse shell over the network, or a hijacked process forking /bin/sh. The built-in 'Terminal shell in container' rule matches proc.name in shell binaries combined with a tty allocation.

---

**Q45. Correct: A**

Kubernetes evaluates audit policy rules in order and applies the FIRST matching rule to each request. Placing a specific rule targeting the exec/attach subresources at RequestResponse level ahead of a broader Metadata-level catch-all rule ensures exec/attach traffic gets full body logging while everything else stays lightweight.

---

**Q46. Correct: C**

Falco's default rules for detecting writes to protected paths (e.g., 'Write below etc', 'Write below binary dir') use evt.type in (open, openat) filtered by evt.is_open_write=true to catch the file being opened with write intent, combined with fd.name matching against a list of sensitive read-only paths. This is a real-time behavioral/syscall detection, not a preventive control.

---

