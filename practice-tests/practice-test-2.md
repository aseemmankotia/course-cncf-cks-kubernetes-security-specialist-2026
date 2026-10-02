# Practice Test 2: CKS: Certified Kubernetes Security Specialist Prep

- **Questions:** 46
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-2.md`. Track results in `score-tracker.md`.

---

**Q1.** You must create a NetworkPolicy in namespace 'payments' that allows ingress traffic ONLY from pods labeled role: frontend that are running inside namespaces labeled team: frontend. Traffic from frontend-labeled pods in any other namespace, or from any pod (without the role label) inside team: frontend namespaces, must be denied. Which policy 'ingress.from' structure correctly enforces this?

A) Two separate items in the 'from' list — one with only a namespaceSelector matching team: frontend, and one with only a podSelector matching role: frontend
B) A single 'from' item combining podSelector: {matchLabels: {role: frontend}} and ipBlock: {cidr: 0.0.0.0/0}
C) A single 'from' item with only namespaceSelector: {matchLabels: {team: frontend}}, relying on the frontend deployment to enforce pod labeling
D) A single 'from' item combining namespaceSelector: {matchLabels: {team: frontend}} and podSelector: {matchLabels: {role: frontend}} in the same object

---

**Q2.** Per the CIS Kubernetes Benchmark, which kubelet configuration correctly closes the anonymous authentication attack surface while still delegating authorization decisions to the API server's RBAC policies?

A) authentication.anonymous.enabled: true and authorization.mode: AlwaysAllow
B) authentication.anonymous.enabled: false and authorization.mode: AlwaysAllow
C) authentication.anonymous.enabled: true and authorization.mode: Webhook
D) authentication.anonymous.enabled: false and authorization.mode: Webhook

---

**Q3.** During a CIS benchmark review of etcd's static pod manifest, you see --cert-file and --key-file are set, but --client-cert-auth and --peer-client-cert-auth are absent. What is the resulting risk, and what is the correct fix?

A) etcd data at rest is stored unencrypted; fix by adding --encryption-provider-config to the API server
B) etcd snapshot backups are world-readable; fix by tightening file permissions on the snapshot directory
C) The API server cannot resolve etcd's advertised endpoints; fix by correcting --etcd-servers on the API server
D) etcd will accept client and peer connections without verifying they present a certificate signed by the trusted CA; fix by setting --client-cert-auth=true and --peer-client-cert-auth=true

---

**Q4.** Which kube-apiserver flag combination should you verify per the CIS Benchmark to ensure the insecure port is disabled and the profiling debug endpoint is turned off?

A) --insecure-port=8080 and --profiling=true
B) --anonymous-auth=true and --insecure-bind-address=0.0.0.0
C) --insecure-port=0 and --profiling=false
D) --enable-admission-plugins=AlwaysAdmit

---

**Q5.** Pods on cloud-hosted worker nodes can successfully query http://169.254.169.254/latest/meta-data/iam/security-credentials/ and retrieve the node's IAM role credentials. Which control BEST mitigates this specific risk at the cluster networking layer?

A) Apply the Pod Security Admission 'restricted' profile to the workload namespaces
B) Rotate the node's IAM role access keys on a daily schedule
C) Set automountServiceAccountToken: false on all workload pod specs
D) Add a NetworkPolicy egress rule denying traffic to 169.254.169.254/32 from workload pod namespaces, combined with enforcing IMDSv2/reduced hop-limit at the cloud provider level

---

**Q6.** You downloaded the kubeadm binary from the official Kubernetes release channel before installing it on a hardened control-plane node. Which method authoritatively verifies BOTH the integrity and the publisher authenticity of the binary before use?

A) Confirm the downloaded file size matches the size listed on the download page
B) Run 'kubeadm version' immediately after install and confirm the version string is correct
C) Confirm the binary executes without throwing an error
D) Verify the published SHA-512 checksum against the downloaded file AND validate the release's signature (e.g., cosign/GPG) against the Kubernetes release signing key

---

**Q7.** A cluster runs on a cloud provider's managed Kubernetes offering. Pods in the `web` namespace must be prevented from reaching the instance metadata service at 169.254.169.254, but they still need DNS resolution (kube-dns/CoreDNS) and general internet egress for legitimate API calls. Which NetworkPolicy design correctly achieves this using only native Kubernetes NetworkPolicy resources?

A) Create a NetworkPolicy with an empty podSelector and policyTypes: Egress, with one egress rule allowing ipBlock 0.0.0.0/0 except 169.254.169.254/32, plus a second egress rule explicitly allowing UDP/TCP port 53 to the DNS pods
B) Create a NetworkPolicy with podSelector matching label app=metadata-blocker and an ingress rule denying traffic from 169.254.169.254/32, applied cluster-wide
C) Deploy a PodSecurityPolicy that sets hostNetwork: false and blocks pods from binding to link-local address ranges
D) Create a NetworkPolicy with an empty podSelector and an egress rule specifying ipBlock cidr 169.254.169.254/32 as the only egress destination, relying on Kubernetes' default-deny behavior for all other traffic

---

**Q8.** A security audit finds that kubelet's read-only port 10255 is reachable from the pod network and returns detailed metrics and pod specs without any credentials. Which remediation correctly eliminates this exposure?

A) Configure a NetworkPolicy to block traffic to port 10255 from all namespaces
B) Set readOnlyPort: 0 in the kubelet configuration file to disable the unauthenticated read-only port entirely
C) Enable authentication on port 10255 by setting --anonymous-auth=false
D) Rotate the kubelet's serving certificate to invalidate existing connections to port 10255

---

**Q9.** You need to harden kubelet's API so that unauthenticated requests are rejected and authorization decisions are delegated to the API server via SubjectAccessReview. Which kubelet configuration achieves this?

A) authentication.anonymous.enabled: true, authorization.mode: AlwaysAllow
B) authentication.anonymous.enabled: false, authorization.mode: AlwaysAllow
C) authentication.x509.clientCAFile: "", authorization.mode: Webhook
D) authentication.anonymous.enabled: false, authorization.mode: Webhook

---

**Q10.** Security policy requires that pods in the 'prod' namespace do NOT receive service account tokens by default, but the 'payment-api' Deployment must retain API access to watch ConfigMaps. Which approach satisfies both requirements with least privilege?

A) Set automountServiceAccountToken: false on the namespace's default ServiceAccount, and set automountServiceAccountToken: true on the payment-api pod spec or its dedicated ServiceAccount
B) Delete the default ServiceAccount in the prod namespace and recreate pods without any serviceAccountName reference
C) Apply a PodSecurityPolicy that denies volume mounts named kube-api-access-* for all pods except payment-api
D) Set automountServiceAccountToken: false only in the payment-api Deployment's pod spec

---

**Q11.** A user's ClusterRole only grants get and list on pods. They attempt to create a ClusterRoleBinding that binds a separate, highly privileged ClusterRole (with * on *) to their own account. Which RBAC verb, if absent on the clusterroles/clusterrolebindings resources for this user, causes the API server's default privilege-escalation prevention to block this action?

A) escalate
B) impersonate
C) approve
D) bind

---

**Q12.** An attacker compromises the kubelet credentials on worker-node-3 and attempts to use them to modify labels on worker-node-7, hoping to reschedule a malicious pod there via manipulated node affinity. Which control specifically prevents this cross-node action even though the compromised identity is a valid, authenticated kubelet?

A) The NodeRestriction admission controller
B) PodSecurity admission configured in 'restricted' mode
C) --authorization-mode=RBAC alone, with no additional admission controllers enabled
D) Enabling audit logging with a RequestResponse level policy on node objects

---

**Q13.** Since Kubernetes 1.24, what is the default behavior of service account tokens mounted into pods via the projected volume mechanism, and why does this reduce risk compared to the legacy pre-1.24 mechanism?

A) Tokens are permanent and stored as long-lived Secrets, identical to pre-1.24 behavior for backward compatibility
B) Tokens are time-bound (default 1 hour), audience-bound, and automatically rotated by the kubelet, limiting the value of a leaked token
C) Tokens are replaced entirely by client certificates issued through the CertificateSigningRequest API
D) Tokens are encrypted at rest using the cluster's etcd encryption provider by default, but retain infinite validity

---

**Q14.** A security audit of a Kubernetes cluster finds that unauthenticated curl requests to https://<node-ip>:10250/pods return full pod manifests, including mounted secret names, with no credentials supplied. Inspecting the kubelet configuration on the node shows authentication.anonymous.enabled: true and authorization.mode: AlwaysAllow. The cluster's API server must continue to reach kubelets on port 10250 for exec, logs, and metrics-server scraping. Which single change BEST remediates the exposure while preserving this required control-plane functionality?

A) Set authentication.anonymous.enabled: false and authorization.mode: Webhook in the kubelet config, then restart the kubelet
B) Remove the --port flag from the kubelet systemd unit to disable the kubelet API entirely
C) Set authentication.anonymous.enabled: false and authorization.mode: AlwaysDeny in the kubelet config
D) Change the kubelet's serving port from 10250 to a non-standard high port and keep anonymous auth enabled

---

**Q15.** A security team must prevent containers from invoking dangerous syscalls such as mount, ptrace, and reboot, without modifying application source code. Which approach BEST meets this requirement on a Kubernetes v1.31 cluster?

A) Enable the AppArmor default (docker-default) profile on the container
B) Create a PodSecurityPolicy that restricts allowedProcMountTypes
C) Apply a custom seccomp profile that filters the syscall table and set seccompProfile.type: Localhost on the Pod
D) Set securityContext.runAsNonRoot: true on the container spec

---

**Q16.** You are hardening worker nodes running Ubuntu 22.04 in a Kubernetes cluster. Which combination of kernel hardening tools provides BOTH mandatory access control over file paths and fine-grained syscall filtering, matching the CKS-recommended approach for this OS?

A) SELinux for file path MAC combined with seccomp for syscall filtering
B) AppArmor combined with the gVisor (runsc) sandboxed runtime only
C) AppArmor for file path MAC combined with seccomp for syscall filtering
D) seccomp combined with a PodSecurityPolicy for syscall filtering

---

**Q17.** You created a custom seccomp profile file named audit.json to restrict syscalls for a Pod. Which combination of file placement and Pod configuration correctly applies this custom profile via the Localhost type?

A) Place the file at /etc/kubernetes/seccomp/audit.json and set seccompProfile.type: RuntimeDefault
B) Place the file at /var/lib/kubelet/seccomp/profiles/audit.json on the node and set seccompProfile: { type: Localhost, localhostProfile: profiles/audit.json }
C) Embed the JSON profile contents directly inside the securityContext.seccompProfile field of the Pod spec
D) Reference the profile using the annotation container.seccomp.security.alpha.kubernetes.io/pod: localhost/audit.json

---

**Q18.** What is the key functional difference between the seccomp profile types RuntimeDefault and Unconfined?

A) RuntimeDefault applies the container runtime's built-in restricted syscall list; Unconfined disables seccomp filtering entirely for that container
B) RuntimeDefault blocks every syscall except read and write; Unconfined allows only an explicit whitelist of syscalls
C) RuntimeDefault requires a custom localhostProfile JSON file; Unconfined always uses the kubelet's default filter
D) RuntimeDefault is only supported for Windows containers, while Unconfined applies solely to Linux containers

---

**Q19.** A team needs SSH access to Kubernetes worker nodes for troubleshooting. Which practice enforces least privilege at the node level, per CKS system hardening guidance?

A) Grant every engineer sudo NOPASSWD access to avoid delays during incident response
B) Issue each engineer an individual non-root SSH key-based account, disable password authentication, and require sudo with command logging for elevated actions
C) Share one root-level SSH private key among the on-call team for simplicity
D) Enable password-based root SSH login and rotate the shared password every 30 days

---

**Q20.** A security team needs new pods created in the 'payments' namespace to be blocked at admission time if they violate the Restricted Pod Security Standard (e.g., run as root or request hostNetwork). Pods currently running in the namespace must NOT be disrupted or evicted. Which action BEST meets these requirements?

A) Label the namespace pod-security.kubernetes.io/enforce=restricted so only newly created pods are evaluated against the Restricted profile; running pods remain unaffected
B) Label the namespace with pod-security.kubernetes.io/warn=restricted only, then wait for admins to manually review the warnings
C) Taint all nodes hosting the payments namespace with NoSchedule and update pod tolerations for compliant workloads only
D) Apply a Kyverno mutate policy that automatically deletes any existing non-compliant pod in the namespace

---

**Q21.** You must run an untrusted, multi-tenant workload with the strongest possible isolation boundary using a Kubernetes RuntimeClass, accepting higher pod startup latency as a tradeoff. Which technology and characteristic correctly describes this choice?

A) Kata Containers — each pod runs inside a lightweight, hardware-virtualized VM (via KVM/QEMU), giving true hardware-level isolation at the cost of higher startup latency
B) gVisor — a user-space kernel intercepts syscalls via ptrace/systrap and provides weaker isolation than a hardware VM boundary
C) AppArmor profile — confines container syscalls using mandatory access control at the kernel level
D) seccomp-bpf profile — filters syscalls per container using a JSON allow/deny list attached to the pod spec

---

**Q22.** You want the API server to reject any new pod in namespace 'test' that violates the Restricted Pod Security Standard, while pods that merely violate the less-strict Baseline standard should only generate a warning to the client (not be blocked). Which pair of namespace labels achieves this?

A) pod-security.kubernetes.io/enforce=baseline and pod-security.kubernetes.io/warn=restricted
B) pod-security.kubernetes.io/audit=restricted and pod-security.kubernetes.io/enforce=baseline
C) pod-security.kubernetes.io/warn=restricted and pod-security.kubernetes.io/enforce=privileged
D) pod-security.kubernetes.io/enforce=restricted and pod-security.kubernetes.io/warn=baseline

---

**Q23.** A Kyverno ClusterPolicy is deployed to prevent pods from running as root. New pods that explicitly set runAsUser: 0 are still successfully created and running as root, but Kyverno reports the violations in its PolicyReport. What is the most likely cause?

A) Kyverno ClusterPolicies cannot validate pod security contexts and only support mutation, so a MutatingWebhookConfiguration is required instead
B) The policy is missing a 'background: true' field, which is required for Kyverno to evaluate any admission request at all
C) ClusterPolicy resources only apply to resources labeled 'kyverno.io/managed=true', and the pods were created without this label
D) The ClusterPolicy's validationFailureAction is set to Audit, which only logs and reports violations instead of blocking the request; it must be changed to Enforce

---

**Q24.** A compliance requirement states that Kubernetes Secret objects must remain confidential even if an attacker obtains a raw copy of the etcd data files (e.g., from a stolen backup). Which control on the kube-apiserver directly satisfies this requirement?

A) Restrict RBAC so only the 'cluster-admin' ClusterRole can run 'kubectl get secrets', preventing unauthorized reads
B) Configure the kube-apiserver with an EncryptionConfiguration file (--encryption-provider-config) using a provider such as aescbc or a KMS plugin so Secret data is encrypted before being persisted in etcd
C) Enable mutual TLS client certificate authentication between etcd peers to prevent unauthorized network access to the datastore
D) Create secrets using 'kubectl create secret generic' instead of applying YAML manifests, since the CLI base64-encodes values automatically

---

**Q25.** Which statement correctly describes how Cilium implements pod-to-pod mTLS/transparent encryption, as distinguished from a traditional sidecar-based service mesh?

A) Cilium requires injecting an Envoy sidecar container into every pod via a mutating webhook, identical to a traditional service mesh, in order to encrypt any pod-to-pod traffic
B) Cilium enforces network policy using eBPF and can transparently encrypt node-to-node traffic with IPsec or WireGuard, without requiring per-application certificate management
C) Cilium mTLS is implemented purely at the application layer — each microservice must load and rotate its own TLS certificate independently
D) Cilium encrypts only cluster DNS lookups by default; all pod-to-pod data traffic remains in plaintext unless a service mesh is layered on top

---

**Q26.** You created a RuntimeClass named 'gvisor' with handler 'runsc' and set spec.runtimeClassName: gvisor on a pod. The pod fails to schedule with an error stating the runtime handler is not supported on the node. What is the MOST likely cause?

A) The RuntimeClass object must be created in the same namespace as the pod
B) containerd on the node has not been configured with a runsc runtime handler entry in its config.toml
C) gVisor requires PodSecurityPolicy to be enabled to function
D) The pod's seccompProfile must be set to RuntimeDefault when using gVisor

---

**Q27.** A cluster enforces Pod Security Admission at the 'restricted' level via the pod-security.kubernetes.io/enforce label on the 'staging' namespace. A CI/CD service account needs to run a one-off debug pod using hostPath in that namespace, without weakening the restricted policy for any other workload in staging. Which approach BEST satisfies this requirement?

A) Add the annotation pod-security.kubernetes.io/exempt: true directly to the debug pod's spec
B) Configure a Pod Security Admission exemption for the specific service account's username in the API server's AdmissionConfiguration
C) Temporarily change the namespace label to pod-security.kubernetes.io/enforce=privileged just for that deployment
D) Create a Kyverno policy exception referencing the pod's labels, since Kyverno overrides built-in Pod Security Admission automatically

---

**Q28.** Your organization requires all pod-to-pod traffic across the cluster to be encrypted in transit, without modifying application code or injecting sidecar proxies. The cluster's CNI is Cilium. Which configuration BEST meets this requirement?

A) Deploy Istio with automatic sidecar injection to provide mTLS between pods
B) Configure Kubernetes NetworkPolicies with TLS termination annotations between pods
C) Enable Cilium's transparent encryption feature using IPsec or WireGuard at the node level
D) Enable etcd encryption at rest so that data in transit between pods is protected

---

**Q29.** Your security team requires that the API server admit only container images from a trusted internal registry, and mandates using a native Kubernetes admission mechanism rather than deploying any third-party controller. Which mechanism should you configure on the API server?

A) OPA Gatekeeper with a ConstraintTemplate restricting allowed image registries
B) A Kubernetes NetworkPolicy restricting egress to registry IP ranges
C) PodSecurityPolicy with the allowedHostPaths field
D) The built-in ImagePolicyWebhook admission controller configured via --admission-control-config-file

---

**Q30.** A CI pipeline stage runs: trivy image --exit-code 1 --severity CRITICAL myapp:v2. The scan finds 3 CRITICAL and 5 HIGH severity vulnerabilities. What happens to the pipeline stage?

A) Exit code 0; pipeline passes because trivy only warns by default regardless of flags
B) Exit code 1; pipeline fails because CRITICAL vulnerabilities matched the severity filter
C) Exit code 1; pipeline fails only if HIGH severity findings also exceed 10
D) Exit code 2; pipeline fails due to a scanner error, not vulnerability findings

---

**Q31.** Your CI pipeline signs container images using Sigstore keyless signing (Fulcio short-lived certificates plus Rekor transparency log entries). Before Kubernetes admits a Pod, you need to cryptographically verify these signatures against expected certificate identity and OIDC issuer. Which tool and command should you use?

A) Docker Content Trust (Notary v1) with a delegated root key
B) kube-bench with the CIS Sigstore profile
C) cosign verify with the --certificate-identity and --certificate-oidc-issuer flags
D) Falco with a custom Sigstore rules file

---

**Q32.** You need to scan a Kubernetes Pod manifest with kubesec for security misconfigurations, but organizational policy prohibits sending manifest data to any external network endpoint. Which approach satisfies this requirement?

A) curl -X POST --data-binary @pod.yaml https://v2.kubesec.io/scan
B) kubectl kubesec scan pod.yaml
C) kubesec scan pod.yaml (using the locally installed binary, with no outbound API call)
D) trivy config pod.yaml --checks kubesec

---

**Q33.** What is the primary supply-chain security benefit of using a 'distroless' base image (e.g., gcr.io/distroless/static) compared to a minimal Alpine-based image?

A) Distroless images are automatically signed and verified by cosign at pull time
B) Distroless images remove the shell, package manager, and other OS utilities, reducing the tools available to an attacker after container compromise
C) Distroless images include a full package manager for easier patching in production
D) Distroless images are functionally identical to Alpine images but with a different name

---

**Q34.** A compliance requirement mandates generating a Software Bill of Materials (SBOM) for every container image, in a standardized schema consumable by both vulnerability scanners and license-compliance tooling. Which CI pipeline command correctly produces this artifact?

A) syft myimage:tag -o cyclonedx-json
B) trivy image --format table myimage:tag
C) grype myimage:tag -o json
D) docker inspect myimage:tag

---

**Q35.** A cluster admin must ensure that only container images from the internal registry `registry.internal.corp` can be deployed to the cluster, and any Pod referencing an image from another registry must be rejected at admission time. Which approach BEST satisfies this requirement?

A) Configure an OPA Gatekeeper ConstraintTemplate and Constraint that inspects `spec.containers[*].image` and denies any image whose repository prefix does not match `registry.internal.corp/`
B) Add a NetworkPolicy that blocks egress traffic to all registries except `registry.internal.corp`
C) Set `imagePullPolicy: IfNotPresent` on every Pod so cached images are reused instead of pulled from external registries
D) Create a Secret of type `kubernetes.io/dockerconfigjson` scoped only to `registry.internal.corp` and reference it via `imagePullSecrets`

---

**Q36.** Before a Pod manifest is committed to the GitOps repository, a security engineer wants a static, offline check of the manifest itself — flagging issues such as missing `runAsNonRoot`, containers running as UID 0, absent resource limits, and mounted hostPath volumes — without needing a running cluster. Which tool is designed specifically for this?

A) trivy image
B) cosign
C) kubesec
D) falco

---

**Q37.** Your organization signs all production images with a cosign key pair and wants an admission-time check that rejects any Pod whose image was NOT signed by the corresponding public key `cosign.pub`, without manually running verification for every deployment. Which single step correctly completes this workflow using cosign as the verification mechanism?

A) Run `cosign generate-key-pair` on every node so the kubelet can independently verify signatures locally before pulling
B) Deploy an admission controller (e.g., a Kyverno `verifyImages` policy or Connaisseur) configured with `cosign.pub` so every incoming Pod's image reference is checked with `cosign verify --key cosign.pub $IMAGE` at admission time
C) Run `cosign sign --key cosign.pub $IMAGE` in the CI pipeline immediately before every deployment
D) Store `cosign.pub` as a Kubernetes Secret and mount it into every container via `envFrom` so the application can self-verify its own image at startup

---

**Q38.** A security engineer wants to detect reverse shell activity in containers using Falco, distinguishing malicious outbound shell connections from legitimate `kubectl exec` debugging sessions that also spawn bash. Which Falco rule condition BEST achieves this?

A) evt.type in (connect) and fd.typechar='4' and proc.name in (shell_binaries)
B) evt.type=execve and proc.name in (bash, sh)
C) spawned_process and proc.name in (bash, sh, nc) and container.id != host
D) evt.type=open and fd.name contains "/dev/tcp"

---

**Q39.** You must deploy Falco for syscall-level runtime detection on GKE nodes running Container-Optimized OS (COS), where kernel headers are unavailable and compiling a kernel module is not possible. Which Falco driver should you configure to retain full behavioral monitoring?

A) The classic kernel module driver (falco.ko)
B) A userspace ptrace-based instrumentation driver
C) The modern eBPF probe (modern_bpf / CO-RE based)
D) The legacy BPF probe that requires kernel headers to compile at load time

---

**Q40.** Your Kubernetes audit policy must log full RequestResponse bodies ONLY for `delete` operations on the `secrets` resource, while logging just `Metadata` for every other request cluster-wide. Which policy correctly achieves this, given that audit rules are evaluated top-down and the first matching rule wins?

A) rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["*"]
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
  verbs: ["delete"]
B) rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
- level: Metadata
  resources:
  - group: ""
    resources: ["*"]
    verbs: ["delete"]
C) rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
  verbs: ["delete"]
- level: Metadata
  resources:
  - group: ""
    resources: ["*"]
D) rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["*"]
  verbs: ["delete"]
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]

---

**Q41.** A Falco 'Terminal shell in container' alert is firing repeatedly from a legitimate CI pipeline that uses `kubectl exec` with bash to perform automated health checks against application pods. Disabling the rule cluster-wide is not acceptable because it would remove detection of genuinely malicious shell access elsewhere. What is the BEST remediation?

A) Create an override/exception rule appending a condition that excludes the specific CI service account or container image performing the health checks, while keeping the base rule active for all other contexts
B) Delete the 'Terminal shell in container' rule from the Falco rules file entirely
C) Raise Falco's global minimum priority so only CRITICAL-severity rules generate alerts
D) Switch Falco to a passive audit-only mode so shell-spawn events are recorded but never alert

---

**Q42.** Which statement about enforcing immutability with `readOnlyRootFilesystem: true` at runtime is CORRECT?

A) readOnlyRootFilesystem only makes the container's root filesystem read-only; volumes such as emptyDir or hostPath mounted into the pod remain writable unless also explicitly mounted with readOnly: true, so Falco rules monitoring writes to those mount paths are still necessary
B) readOnlyRootFilesystem prevents all writes within the container, including to mounted emptyDir and hostPath volumes, making Falco write-detection redundant
C) readOnlyRootFilesystem only takes effect if a matching seccomp profile is also applied; otherwise the container runtime ignores the setting
D) Setting readOnlyRootFilesystem: true automatically also sets allowPrivilegeEscalation: false as an implied secure default

---

**Q43.** While reviewing Kubernetes audit logs for a `pods/exec` request, you see multiple entries with different `stage` values: RequestReceived, ResponseStarted, and ResponseComplete. To detect long-running interactive exec sessions potentially used for lateral movement, you need the record containing the final outcome and total session duration. Which audit stage should you filter on?

A) RequestReceived
B) ResponseStarted
C) ResponseComplete
D) Panic

---

**Q44.** A security team wants Falco to alert only when an interactive shell is spawned inside a container by an attacker (not when a container's entrypoint script legitimately invokes /bin/sh non-interactively). Which detection logic BEST achieves this while minimizing false positives?

A) Match evt.type=execve where proc.name is in the shell_binaries list and the process is running inside a container, with no additional filters
B) Match spawned_process events where proc.name is a shell binary, container.id is not host, AND proc.tty is not equal to 0 (i.e., a real TTY is attached), mirroring Falco's built-in 'Terminal shell in container' rule
C) Forward kube-apiserver audit logs to Falcosidekick and alert on every 'pods/exec' subresource create event
D) Apply a seccomp profile that blocks the execve syscall for /bin/sh and /bin/bash inside all containers

---

**Q45.** You must configure an audit policy so that: (1) reads of Secret objects by system:kube-scheduler are never logged, (2) all other access to Secret objects is logged at RequestResponse level, and (3) everything else in the cluster is logged at Metadata level. Rule order in the policy file is:
1. level: RequestResponse, resources: [secrets]
2. level: None, users: [system:kube-scheduler], resources: [secrets]
3. level: Metadata, resources: [*]
Will this policy behave as intended?

A) Yes, because Kubernetes evaluates all matching rules and applies the most restrictive level automatically
B) No, because 'level: None' is not a valid audit level and must be replaced with 'level: Metadata' to suppress logging
C) Yes, because resource-scoped rules always take precedence over user-scoped rules regardless of position in the file
D) No, because audit policy rules are evaluated top-down and the first matching rule wins — the RequestResponse rule for secrets matches kube-scheduler's requests before the None rule is ever reached, so its access still gets fully logged

---

**Q46.** A pod runs with securityContext.readOnlyRootFilesystem: true, but mounts an emptyDir volume at /tmp that remains writable. An attacker compromises the app, writes a malicious ELF binary to /tmp/.hidden/update, and then executes it to establish persistence. Which Falco detection capability is specifically designed to catch this exact behavior pattern?

A) The built-in 'Unexpected outbound connection destination' rule, which flags network calls to IPs outside an allowed list
B) The built-in 'Delete or rename shell history' rule, which detects anti-forensic behavior on bash_history files
C) The built-in 'Drop and execute new binary in container' rule, which fires when a process executes a binary from a location that was recently written to inside the container (rather than shipped in the original image)
D) The built-in 'Contact K8s API Server from container' rule, which flags containers making direct calls to the Kubernetes API

---

