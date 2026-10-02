# Answer Key — Practice Test 2

**Q1. Correct: D**

When namespaceSelector and podSelector appear inside the SAME 'from' entry, Kubernetes ANDs them — only pods matching both the pod label AND running in a matching namespace are allowed. This is the only structure that matches the exact requirement.

---

**Q2. Correct: D**

CIS 4.2.1 requires anonymous-auth=false to prevent unauthenticated requests to the kubelet API, and 4.2.2 requires authorization-mode=Webhook so the kubelet defers every authorization decision to the API server's configured RBAC/webhook authorizer instead of trusting all callers.

---

**Q3. Correct: D**

Without --client-cert-auth and --peer-client-cert-auth, TLS may encrypt the channel but etcd does not require or verify client/peer certificates, meaning any process that can reach the etcd port can authenticate as a trusted member or client — a critical CIS control (etcd section) failure.

---

**Q4. Correct: C**

CIS requires --insecure-port=0 (the insecure port is already removed by default in current Kubernetes but must be verified as 0/absent) and --profiling=false to prevent exposing pprof debug/profiling data that can leak sensitive runtime information.

---

**Q5. Correct: D**

Blocking egress to the instance metadata IP via NetworkPolicy (and hardening the metadata service itself, e.g., requiring IMDSv2 tokens with a low hop-limit so containerized processes can't reach it) directly stops pods from reaching node-level cloud credentials — this is the standard CKS mitigation for the metadata-endpoint attack path.

---

**Q6. Correct: D**

SHA-512 checksum verification confirms the file wasn't corrupted or tampered with in transit, while validating the cryptographic signature against the known Kubernetes release signing key confirms the binary genuinely originated from the official Kubernetes release process — together they cover both integrity and authenticity, which the CKS exam explicitly tests.

---

**Q7. Correct: A**

NetworkPolicy has no explicit 'deny' action — it is purely allow-list based. To block the metadata endpoint while permitting everything else, you must scope an egress rule to 0.0.0.0/0 with an 'except' clause carving out 169.254.169.254/32, and add a separate rule explicitly allowing port 53 to CoreDNS since selecting Egress as a policyType makes the namespace default-deny for anything not explicitly allowed.

---

**Q8. Correct: B**

The read-only port (10255) has no authentication or authorization support whatsoever. The only correct fix is to disable it entirely (readOnlyPort: 0), forcing all API/metrics access through the secure port 10250 where authentication and authorization apply.

---

**Q9. Correct: D**

Disabling anonymous auth forces every request to be authenticated, and authorization.mode: Webhook delegates the access decision to the API server (via SubjectAccessReview), enforcing RBAC-based least privilege on the kubelet API.

---

**Q10. Correct: A**

Kubernetes evaluates automountServiceAccountToken from the most specific scope: a pod-level (or dedicated ServiceAccount-level) setting of true overrides a namespace default ServiceAccount setting of false. This disables token mounting cluster-wide by default while explicitly re-enabling it only where needed.

---

**Q11. Correct: D**

The 'bind' verb specifically governs whether a subject can create a RoleBinding or ClusterRoleBinding that references a Role/ClusterRole granting permissions the subject does not already possess. Without 'bind' on roles/clusterroles, the API server's built-in escalation check rejects the binding.

---

**Q12. Correct: A**

The NodeRestriction admission controller limits kubelets (identities in the system:nodes group) to only modifying their own Node object and Pods bound to their own node, blocking any attempt to alter objects belonging to other nodes.

---

**Q13. Correct: B**

Bound service account tokens, requested via the TokenRequest API and mounted through a projected volume, are short-lived (1 hour default), scoped to specific audiences, and automatically refreshed by the kubelet before expiry — sharply reducing the blast radius if a token is exfiltrated, unlike the old non-expiring Secret-mounted tokens.

---

**Q14. Correct: A**

Disabling anonymous authentication forces every request to present valid credentials, and switching authorization.mode to Webhook delegates authorization decisions to the API server (via SubjectAccessReview), which already holds a valid client certificate and RBAC identity to interact with kubelets for exec, logs, and metrics scraping. This closes the anonymous access hole without breaking legitimate API-server-to-kubelet traffic.

---

**Q15. Correct: C**

Seccomp operates at the syscall level and can explicitly allow or deny individual syscalls like mount, ptrace, and reboot via a JSON profile referenced with seccompProfile.type: Localhost — this is the purpose-built tool for syscall filtering.

---

**Q16. Correct: C**

Ubuntu ships AppArmor as its default LSM for path-based mandatory access control, and seccomp is the standard Kubernetes mechanism for syscall filtering — together they are the CKS-recommended pairing for Debian/Ubuntu-based nodes.

---

**Q17. Correct: B**

The kubelet's seccomp profile root directory is <kubelet-root-dir>/seccomp (default /var/lib/kubelet/seccomp/). The localhostProfile field is a path relative to that root, so the file must be placed under profiles/audit.json and referenced with type Localhost.

---

**Q18. Correct: A**

RuntimeDefault instructs the container runtime (e.g., containerd, CRI-O) to apply its bundled default seccomp filter, which blocks known-dangerous syscalls. Unconfined disables seccomp filtering for the container completely, allowing all syscalls.

---

**Q19. Correct: B**

Per-user key-based non-root accounts with disabled password auth and logged sudo escalation provide traceability and least privilege — each action is attributable to an individual, and elevated access is explicit and auditable.

---

**Q20. Correct: A**

Pod Security Admission (PSA) is evaluated only at admission time (create/update requests). Setting the 'enforce' mode to 'restricted' blocks new non-compliant pods from being created but never inspects or evicts already-running pods, satisfying both requirements exactly.

---

**Q21. Correct: A**

Kata Containers boots each pod inside its own lightweight VM backed by hardware virtualization extensions, providing a true hardware-enforced isolation boundary between the container and the host kernel — the strongest isolation of the RuntimeClass sandbox options, at the cost of slower pod startup than runc or gVisor.

---

**Q22. Correct: D**

Setting enforce=restricted blocks any pod that fails the Restricted profile. Setting warn=baseline separately causes the API server to emit a client-side warning for anything that fails the looser Baseline profile (which restricted pods will already satisfy), without blocking it — matching the requirement exactly.

---

**Q23. Correct: D**

Kyverno validate rules support two failure actions: 'Audit' (report violations via PolicyReport but allow the request) and 'Enforce' (reject the admission request). PolicyReport entries with pods still running as root is the textbook symptom of validationFailureAction being left at the default 'Audit' instead of 'Enforce'.

---

**Q24. Correct: B**

By default, Kubernetes stores Secret data in etcd only base64-encoded (not encrypted). Configuring encryption-at-rest via an EncryptionConfiguration passed with --encryption-provider-config causes the API server to encrypt Secret payloads (e.g., with AES-CBC or an external KMS) before writing them to etcd, so stolen etcd data files remain unreadable without the encryption key.

---

**Q25. Correct: B**

Cilium's CNI uses eBPF programs attached at the kernel level to enforce network policy and can enable transparent encryption of traffic between nodes using IPsec or WireGuard tunnels. This happens below the application layer, so individual microservices do not need to manage their own TLS certificates for this transport-level protection — a key distinction from sidecar-proxy service meshes like Istio.

---

**Q26. Correct: B**

RuntimeClass only tells Kubernetes which handler name to request; the container runtime (containerd/CRI-O) on each node must have that handler (e.g. runsc for gVisor) registered in its configuration. If the node's containerd config.toml lacks the runsc plugin entry, scheduling/creation fails with an unsupported runtime handler error.

---

**Q27. Correct: B**

Pod Security Admission supports exemptions configured cluster-wide in the AdmissionConfiguration (via --admission-control-config-file), which can list specific usernames, runtimeClassNames, or namespaces to bypass enforcement. Exempting the CI/CD service account's username lets that identity create the debug pod without touching the namespace-wide 'restricted' label or affecting other workloads.

---

**Q28. Correct: C**

Cilium natively supports transparent encryption of pod-to-pod traffic using either IPsec or WireGuard, implemented at the node/network layer via eBPF. This encrypts traffic between nodes/pods without requiring any application changes or sidecar proxy injection, directly meeting the stated constraints.

---

**Q29. Correct: D**

ImagePolicyWebhook is a built-in kube-apiserver admission plugin enabled with --enable-admission-plugins=ImagePolicyWebhook and configured via an AdmissionConfiguration file pointing to a webhook backend that decides whether an image reference is allowed. It requires no third-party controller to be installed in the cluster.

---

**Q30. Correct: B**

The --severity CRITICAL flag limits which findings trivy counts, and --exit-code 1 tells trivy to return exit code 1 if any vulnerabilities matching that filter are found. Since 3 CRITICAL vulnerabilities were found, trivy exits with 1 and most CI systems treat a non-zero exit code as a failed stage.

---

**Q31. Correct: C**

cosign is the Sigstore project's client tool and natively supports keyless verification. cosign verify --certificate-identity=<expected-signer> --certificate-oidc-issuer=<expected-issuer> <image> checks the Fulcio-issued certificate embedded in the signature and confirms the Rekor transparency log entry, confirming the image was signed by the expected CI identity.

---

**Q32. Correct: C**

kubesec ships as a standalone Go binary (and container image) that evaluates manifests entirely locally, producing a JSON risk score with no network calls required — satisfying an offline/air-gapped policy constraint.

---

**Q33. Correct: B**

Distroless images strip out shells, package managers (apk/apt), and most coreutils, leaving only the application and its runtime dependencies. Even if an attacker exploits an application vulnerability and gains code execution, there is no shell or package manager available to pivot, install tools, or explore the filesystem interactively — significantly reducing post-compromise capability.

---

**Q34. Correct: A**

syft is purpose-built to generate SBOMs by cataloging packages, libraries, and metadata inside an image, and -o cyclonedx-json emits it in the CycloneDX standard schema — an industry format understood by downstream vulnerability scanners (e.g., grype) and license-compliance tools.

---

**Q35. Correct: A**

OPA Gatekeeper (or an equivalent admission controller like Kyverno/ImagePolicyWebhook) evaluates the Pod spec's image field at admission time and can reject the request outright if the registry prefix doesn't match an allowed list — this is the standard CKS-tested pattern for enforcing allowed registries.

---

**Q36. Correct: C**

kubesec performs static analysis directly on Kubernetes YAML manifests, scoring them against known security risk factors (root user, privileged mode, missing limits, hostPath mounts, etc.) entirely offline, which matches exactly the described workflow and is explicitly named in the CKS curriculum's supply chain security domain.

---

**Q37. Correct: B**

Since cosign itself has no native Kubernetes admission hook, the CKS-tested pattern is to integrate cosign verification into an admission controller (Kyverno's verifyImages, Connaisseur, or similar) configured with the public key so every Pod's image is verified automatically at admission time — exactly satisfying 'without manually running verification for every deployment'.

---

**Q38. Correct: A**

Combining a network connect syscall (fd.typechar='4' for IPv4 sockets) with process ancestry in shell_binaries matches the actual behavioral pattern of a reverse shell: a shell process initiating an outbound TCP connection. This is the pattern Falco's built-in 'Reverse Shell' style rules use, and it correctly ignores kubectl exec sessions that spawn bash without the process itself opening an outbound socket.

---

**Q39. Correct: C**

The modern eBPF probe uses CO-RE (Compile Once – Run Everywhere) with BTF, so it does not require kernel headers or on-node compilation, making it the correct driver choice for locked-down COS nodes on GKE.

---

**Q40. Correct: C**

Placing the specific, narrowly-scoped rule (secrets + delete verb, RequestResponse) first ensures it is matched before the catch-all wildcard rule. The wildcard Metadata rule with no verb restriction then correctly captures everything else, satisfying both requirements.

---

**Q41. Correct: A**

Falco supports rule overrides and exceptions (via `append: true` custom rule files or the `exceptions` field) that add a targeted `and not` condition scoped to the known-good CI identity or image, preserving detection coverage for every other shell-spawn scenario in the cluster.

---

**Q42. Correct: A**

readOnlyRootFilesystem is enforced by the container runtime purely against the container's root filesystem layer; any volume explicitly mounted into the pod (emptyDir, hostPath, PVC, etc.) is independently writable unless that specific volumeMount also sets readOnly: true. This is a classic CKS exam trap — engineers assume root-fs immutability is total, leaving writable attack surfaces that Falco must still monitor.

---

**Q43. Correct: C**

ResponseComplete is recorded once the response body has finished sending and represents the terminal record of the request lifecycle, containing full metadata and enabling duration calculation from requestReceivedTimestamp to stageTimestamp — this is the correct stage for identifying completed exec session outcomes.

---

**Q44. Correct: B**

The proc.tty != 0 condition is the key discriminator: an interactively attached shell (e.g., via a reverse shell or attacker exec) has a real TTY, while shells invoked programmatically by an entrypoint script or init process do not. This is exactly the logic Falco's default 'Terminal shell in container' rule uses, giving high-fidelity detection with low false positives.

---

**Q45. Correct: D**

Kubernetes audit policy rules are evaluated in list order, and the FIRST rule that matches a given request determines its audit level — subsequent rules are never consulted for that request. Since the RequestResponse rule for secrets is listed before the None rule scoped to system:kube-scheduler, every secrets request (including kube-scheduler's) matches rule 1 first and is logged at RequestResponse, defeating the intended exclusion. The None rule must be moved above the RequestResponse rule to work.

---

**Q46. Correct: C**

Falco's 'Drop and execute new binary in container' rule is purpose-built to detect the write-then-execute pattern: it correlates a recent write to a file path with a subsequent execve of that same binary inside the container, which is precisely how an attacker abuses a writable volume (like /tmp) alongside an otherwise read-only root filesystem to plant and run malware.

---

