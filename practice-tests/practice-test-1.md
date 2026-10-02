# Practice Test 1: CKS: Certified Kubernetes Security Specialist Prep

- **Questions:** 46
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-1.md`. Track results in `score-tracker.md`.

---

**Q1.** A company needs pods in namespace 'finance' to accept ingress traffic ONLY from pods labeled role=reporting that live in namespace 'analytics'. All other ingress traffic to 'finance' pods must be blocked. Which NetworkPolicy correctly implements this?

A) apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-reporting
  namespace: finance
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: analytics}
    - podSelector:
        matchLabels: {role: reporting}
B) apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-reporting
  namespace: finance
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: analytics}
      podSelector:
        matchLabels: {role: reporting}
C) apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-reporting
  namespace: finance
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: analytics}
      podSelector:
        matchLabels: {role: reporting}
D) apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-reporting
  namespace: analytics
spec:
  podSelector:
    matchLabels: {role: reporting}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: finance}

---

**Q2.** kube-bench reports FAIL for etcd check '2.2 Ensure that the --client-cert-auth argument is set to true'. Which change to the etcd static pod manifest remediates this finding?

A) Add '--peer-client-cert-auth=true' to the etcd command arguments
B) Add '--client-cert-auth=true' to the etcd command arguments
C) Set '--auto-tls=true' to auto-generate client certificates
D) Add '--cert-file' and '--key-file' arguments only, without any cert-auth flag

---

**Q3.** Which kubelet configuration correctly disables anonymous authentication and enforces webhook-based authorization, matching CIS Kubernetes Benchmark recommendations for kubelet hardening?

A) authentication:
  anonymous:
    enabled: true
authorization:
  mode: Webhook
B) authentication:
  anonymous:
    enabled: false
authorization:
  mode: AlwaysAllow
C) authentication:
  anonymous:
    enabled: false
authorization:
  mode: Webhook
D) authorization:
  mode: Webhook

---

**Q4.** Which Kubernetes resource must exist in the cluster BEFORE it can be referenced in an Ingress object's 'spec.tls[].secretName' field to terminate TLS at the ingress controller?

A) A ConfigMap containing the base64-encoded certificate and key
B) A Secret of type Opaque containing a single 'cert' key
C) A CertificateSigningRequest (CSR) object approved by the cluster CA
D) A Secret of type kubernetes.io/tls containing tls.crt and tls.key

---

**Q5.** On a cloud-hosted Kubernetes cluster, pods can potentially reach the instance metadata service at 169.254.169.254 and retrieve the node's cloud IAM credentials. Which control most directly and specifically mitigates this exposure?

A) Apply Pod Security Admission 'restricted' profile to the namespace
B) Create a NetworkPolicy that denies egress traffic from pods to 169.254.169.254
C) Restrict RBAC access to the 'nodes/proxy' subresource
D) Apply a default seccomp profile of RuntimeDefault to all pods

---

**Q6.** Before installing a newly downloaded kubeadm binary onto a cluster node, which step verifies that the binary has not been tampered with or corrupted in transit?

A) Run 'kubeadm version' immediately after copying it to the node
B) Confirm the downloaded file size roughly matches the previous kubeadm version
C) Scan the binary using kube-bench before installation
D) Compare the SHA-256 checksum of the downloaded binary against the checksum published on the official Kubernetes release page

---

**Q7.** A cluster runs on a cloud provider where nodes can query the instance metadata service at 169.254.169.254 to retrieve IAM credentials. Security auditors find that any pod in the cluster can reach this endpoint and extract node-level cloud credentials. Which solution BEST prevents pods from reaching the metadata endpoint while still allowing normal pod-to-pod and pod-to-internet traffic?

A) Disable the kubelet read-only port on every node so pods cannot query metadata
B) Enable audit logging on the API server to record which pods access the metadata endpoint
C) Rotate the cloud provider IAM credentials on a daily schedule using a CronJob
D) Apply a NetworkPolicy in each namespace that denies egress traffic to 169.254.169.254/32 for all pods except those explicitly requiring cloud API access

---

**Q8.** A security team wants pods running in the "default" namespace to stop automatically receiving a mounted service account token unless a workload explicitly requires API access. Which approach correctly enforces this while keeping the cluster functional?

A) Create a NetworkPolicy denying access to the Kubernetes API server
B) Delete the default service account from the namespace
C) Set enableServiceLinks: false in the pod spec
D) Set automountServiceAccountToken: false on the ServiceAccount or Pod spec

---

**Q9.** You must configure the kube-apiserver so that it rejects all unauthenticated (anonymous) requests, forcing every client to present valid credentials. Which flag directly accomplishes this?

A) --anonymous-auth=false
B) --insecure-port=0
C) --authorization-mode=Node,RBAC
D) --enable-admission-plugins=NodeRestriction

---

**Q10.** During a cluster hardening review, you find the kubelet API accepts requests without any credentials. Which configuration change stops the kubelet from serving unauthenticated (anonymous) requests?

A) authentication.anonymous.enabled: false in the kubelet config file
B) --read-only-port=10250
C) --authorization-mode=AlwaysAllow
D) --protect-kernel-defaults=true

---

**Q11.** A cluster administrator needs to grant a CI/CD pipeline's ServiceAccount permission to only get and list Pods within the "build" namespace, following the principle of least privilege. Which combination of objects should be created?

A) ClusterRole bound via a ClusterRoleBinding
B) A ServiceAccount granted the cluster-admin ClusterRole
C) A PodSecurityPolicy restricting pod permissions
D) Role bound via a RoleBinding scoped to the build namespace

---

**Q12.** What is the primary security purpose of the NodeRestriction admission controller in a hardened Kubernetes cluster?

A) It encrypts Secret objects at rest using envelope encryption
B) It restricts which container images nodes are permitted to pull
C) It limits the Node and Pod objects a kubelet can modify to those it hosts
D) It enforces NetworkPolicy rules at the node level

---

**Q13.** You must ensure etcd only accepts client connections from callers presenting a valid client certificate signed by the cluster's trusted CA before granting access to cluster state. Which etcd flag enforces this?

A) --client-cert-auth=true
B) --peer-client-cert-auth=true
C) --auto-tls=true
D) --cert-file=/etc/etcd/server.crt

---

**Q14.** During a CKS lab, you discover that the kubelet on all worker nodes accepts unauthenticated requests and permits any authenticated user to invoke any kubelet API operation, including exec and logs retrieval. Which kubelet configuration change BEST remediates this vulnerability?

A) Set --anonymous-auth=true and --authorization-mode=AlwaysAllow on the kubelet
B) Set --anonymous-auth=false and --authorization-mode=Webhook on the kubelet
C) Remove the kubelet systemd service so port 10250 is no longer reachable
D) Set --read-only-port=10255 on the kubelet to restrict access to authenticated users only

---

**Q15.** A security engineer must restrict a Pod named 'payment-api' so that it runs with a custom AppArmor profile named 'k8s-payment-restrict' that has already been loaded on every node. Which configuration correctly enforces this in a cluster running Kubernetes 1.30+?

A) Add the annotation container.apparmor.security.beta.kubernetes.io/payment-api: localhost/k8s-payment-restrict to the Pod metadata only
B) Set securityContext.appArmorProfile.type: Localhost and securityContext.appArmorProfile.localhostProfile: k8s-payment-restrict under the container's securityContext
C) Set securityContext.seccompProfile.type: Localhost and localhostProfile: k8s-payment-restrict under the Pod spec
D) Create a PodSecurityPolicy with the annotation apparmor.security.beta.kubernetes.io/allowedProfileNames set to k8s-payment-restrict

---

**Q16.** A container in a cluster must be prevented from making any syscall not explicitly allowed by a custom JSON profile stored at /var/lib/kubelet/seccomp/profiles/audit.json on each node. Which securityContext configuration achieves this?

A) seccompProfile: { type: RuntimeDefault }
B) seccompProfile: { type: Unconfined }
C) appArmorProfile: { type: Localhost, localhostProfile: profiles/audit.json }
D) seccompProfile: { type: Localhost, localhostProfile: profiles/audit.json }

---

**Q17.** As part of reducing the host OS attack surface on worker nodes before joining them to a Kubernetes cluster, which action provides the MOST direct reduction in exploitable attack surface?

A) Enable audit logging on the API server
B) Enable RBAC on the cluster
C) Install a minimal, purpose-built OS (e.g., a container-optimized distribution) and remove unused kernel modules and services
D) Deploy a NetworkPolicy denying all ingress traffic to workloads

---

**Q18.** A cluster administrator wants worker nodes to reject any inbound SSH connection attempts except from a dedicated bastion host, as part of minimizing external access to nodes. Which is the BEST approach?

A) Configure a NetworkPolicy in the kube-system namespace restricting port 22
B) Set the kubelet flag --protect-kernel-defaults=true on every node
C) Configure host-level firewall rules (e.g., iptables/nftables or a cloud security group) that allow port 22 only from the bastion host's IP and deny it from all other sources
D) Create a Kubernetes Service of type NodePort restricted to the bastion host's IP range

---

**Q19.** Which statement BEST describes the difference between AppArmor and seccomp as kernel hardening tools tested on the CKS exam?

A) AppArmor confines a process's access to files, network, and capabilities using path-based mandatory access control profiles, while seccomp filters which syscalls a process is permitted to invoke
B) AppArmor and seccomp are interchangeable; either can enforce identical restrictions on file access and syscalls
C) AppArmor restricts syscalls a process may invoke, while seccomp restricts which files, capabilities, and network resources a process may access
D) AppArmor only works on container images, while seccomp only works on the host OS outside containers

---

**Q20.** A platform team wants to enforce the 'restricted' Pod Security Standard on the 'payments' namespace without deploying any third-party admission controller, while first auditing violations before full enforcement. Which solution BEST meets these requirements?

A) Apply only the label pod-security.kubernetes.io/enforce=restricted to the namespace
B) Install OPA Gatekeeper and create a restricted ConstraintTemplate for the namespace
C) Deploy Kyverno with a cluster policy set to validationFailureAction: Enforce
D) Apply labels pod-security.kubernetes.io/audit=restricted and pod-security.kubernetes.io/warn=restricted first, then add enforce=restricted once violations are resolved

---

**Q21.** A team must run an untrusted, multi-tenant workload requiring isolation via a user-space kernel that intercepts and emulates syscalls, while remaining compatible with the existing containerd CRI shim v2 interface. Which solution BEST fits?

A) Kata Containers with a lightweight VM per pod
B) A custom AppArmor profile applied to the pod
C) The default seccomp profile enabled on the kubelet
D) gVisor (runsc) as the OCI runtime

---

**Q22.** You need a Kyverno ClusterPolicy that blocks admission of any Pod not running as non-root. Which policy definition correctly achieves this?

A) validationFailureAction: Enforce, with a validate rule pattern requiring securityContext.runAsNonRoot: true
B) validationFailureAction: Audit, with a validate rule pattern requiring securityContext.runAsNonRoot: true
C) validationFailureAction: Enforce, using a verifyImages rule to check runAsNonRoot
D) validationFailureAction: Enforce, using a mutate rule to set runAsNonRoot: true on incoming Pods

---

**Q23.** What is the primary architectural difference between OPA Gatekeeper and Kyverno for enforcing Kubernetes admission policies?

A) Gatekeeper defines policies using the Rego language via ConstraintTemplates and Constraints, while Kyverno defines policies as native Kubernetes-style YAML resources without a separate query language
B) Gatekeeper only supports mutating webhooks, while Kyverno only supports validating webhooks
C) Kyverno requires the Rego policy language, while Gatekeeper uses plain YAML
D) Gatekeeper cannot audit existing resources for compliance, while only Kyverno supports an audit mode

---

**Q24.** A security audit finds that Kubernetes Secrets are stored as base64-encoded plaintext in etcd with no encryption. Which solution BEST addresses this at the cluster level?

A) Configure the kube-apiserver with an EncryptionConfiguration using an aescbc or kms provider, then re-encrypt existing Secrets
B) Base64-encode the Secret values a second time before creating them
C) Store sensitive values as ConfigMaps encrypted at the application layer instead of Secrets
D) Restrict access to Secret objects using RBAC Role and RoleBinding resources

---

**Q25.** A cluster requires automatic, transparent mutual TLS encryption of all pod-to-pod traffic across nodes, without modifying application code or injecting sidecar proxies. Which solution BEST meets this requirement?

A) Deploy Istio service mesh with automatic sidecar injection for mTLS
B) Create NetworkPolicy objects to restrict traffic between namespaces
C) Enable Cilium's transparent encryption using WireGuard or IPsec between nodes
D) Use cert-manager to issue a TLS certificate to each Pod

---

**Q26.** A platform team wants to block any Pod from being created in a Kubernetes cluster if it lacks a resource limit definition, enforced via Kyverno (already installed and running). Which Kyverno resource must be created to enforce this restriction cluster-wide?

A) A ClusterPolicy resource containing a validate rule with a pattern requiring resource limits
B) A ValidatingWebhookConfiguration manually pointed at the Kyverno admission service
C) A NetworkPolicy resource restricting egress traffic from all Pods to the API server
D) A PodSecurityPolicy object that sets default resource limits for all namespaces

---

**Q27.** Which container runtime sandbox intercepts container syscalls in a user-space kernel process rather than relying on full hardware virtualization, and is typically wired into Kubernetes with a RuntimeClass referencing the 'runsc' handler?

A) gVisor
B) Kata Containers
C) Firecracker microVMs
D) runc (default OCI runtime)

---

**Q28.** A company must ensure Secret objects are encrypted at rest in etcd using AES-CBC on a kubeadm-managed control plane. Which configuration correctly enables this?

A) Set `--encrypt-secret-data=true` in the kube-controller-manager static pod manifest
B) Create a Kubernetes Secret named `encryption-config` and annotate the etcd StatefulSet to reference it
C) Run `kubectl edit secrets --encrypt-at-rest` in each namespace that stores sensitive Secrets
D) Add `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml` to the kube-apiserver manifest, referencing an EncryptionConfiguration resource with an aescbc provider

---

**Q29.** A company wants to minimize the attack surface of container images used in its Kubernetes cluster. The current base images use full Ubuntu with a shell, package manager, and many unused utilities. The application is not statically compiled and still requires a minimal runtime. Which approach BEST reduces the attack surface while keeping the application functional?

A) Continue using the Ubuntu base image but run apt-get autoremove and apt-get clean at the end of the build
B) Switch to a distroless base image containing only the language runtime and application dependencies
C) Use Alpine Linux and manually delete /bin/sh from the final image layer
D) Use the scratch base image for all applications regardless of language runtime requirements

---

**Q30.** A security engineer wants to run static analysis directly on Dockerfiles before the image is built, flagging issues such as running as root, using ADD instead of COPY, and missing a USER instruction. Which tool is purpose-built for this task?

A) hadolint
B) kubesec
C) trivy
D) kube-bench

---

**Q31.** You must ensure that only images from `registry.corp.io` can be deployed anywhere in the cluster, rejecting Pods that reference any other registry at admission time. Which mechanism should you configure?

A) A NetworkPolicy that restricts egress traffic to registry.corp.io
B) A PodSecurityPolicy with an allowedRegistries field
C) ImagePullSecrets scoped only to registry.corp.io on the default ServiceAccount
D) An OPA Gatekeeper (or ValidatingAdmissionPolicy) constraint that validates the image field prefix on pod creation

---

**Q32.** A CI/CD pipeline must fail the build immediately if the container image contains any CRITICAL severity CVEs before the image is pushed to the registry. Which approach BEST accomplishes this?

A) Run kube-bench against the freshly built image in the pipeline
B) Run a trivy image scan with --severity CRITICAL --exit-code 1 as a pipeline step
C) Configure a NetworkPolicy to block all outbound traffic during the build stage
D) Run a kubesec scan against the application's Deployment manifest

---

**Q33.** Which statement correctly differentiates the two most common SBOM (Software Bill of Materials) formats referenced in Kubernetes supply chain security tooling?

A) SPDX is a container runtime and CycloneDX is a Kubernetes admission controller used to enforce SBOM policy
B) SPDX and CycloneDX are both SBOM standards; SPDX originated in software license compliance while CycloneDX was designed with a stronger focus on security use cases
C) SBOM formats only record the results of a vulnerability scan and never list software components or dependencies
D) CycloneDX has been deprecated in favor of SPDX as the sole format supported by supply chain tools

---

**Q34.** An organization wants to guarantee that only container images signed by its internal CI pipeline using cosign are permitted to run in the cluster, with unsigned or improperly signed images rejected at deploy time. Which combination of steps BEST achieves this?

A) Sign images with `cosign sign` in the CI pipeline, then deploy a policy engine such as Kyverno or Sigstore Policy Controller configured with the public key to verify signatures at admission time
B) Store the cosign public key as a Kubernetes Secret and rely on the kubelet to automatically verify image signatures before pulling
C) Sign images using `docker trust sign` and enable Docker Content Trust on every worker node
D) Add an annotation referencing the cosign public key directly on the Pod spec

---

**Q35.** A platform team must guarantee that only container images pulled from the company's internal Harbor registry (registry.internal.corp) can ever be scheduled onto the cluster, blocking any image reference from Docker Hub or other public registries at admission time. Which solution BEST meets this requirement?

A) Create a NetworkPolicy that denies egress traffic from all nodes to any IP address except the internal registry's IP range
B) Deploy an admission controller (e.g., OPA Gatekeeper or ImagePolicyWebhook) with a policy that rejects any Pod whose container image reference does not match the registry.internal.corp prefix
C) Configure RBAC so that only the platform-admins group has permission to run 'kubectl create deployment', restricting who can launch new workloads
D) Set imagePullPolicy: Always on every Pod spec so the kubelet always re-pulls the latest image from the internal registry

---

**Q36.** You need to statically analyze a Kubernetes Deployment YAML manifest before it is applied, producing a numeric risk score based on checks such as running as root, missing resource limits, and use of hostPath volumes. Which tool is purpose-built for this?

A) Clair
B) kubesec
C) Falco
D) Anchore Engine

---

**Q37.** Your organization signs all production images with cosign and stores the signatures alongside the images in the registry. You must ensure the cluster refuses to run any Pod whose image cannot be cryptographically verified against the trusted public key before admission. Which approach BEST achieves this?

A) Add a livenessProbe that runs 'cosign verify' inside the container after it starts, and let Kubernetes restart the Pod if the check fails
B) Deploy a policy-enforcement admission webhook (such as Sigstore's Policy Controller / Connaisseur) configured with the trusted public key, so unsigned or invalid-signature images are rejected at the API server before scheduling
C) Rely on the registry's TLS certificate to guarantee that only signed images were pushed, since TLS in transit implies integrity
D) Enable imagePullSecrets on the namespace so only authenticated pulls from the private registry are permitted

---

**Q38.** A security team suspects an attacker has gained a foothold in a production pod and is attempting to open an interactive shell to explore the filesystem. They want real-time detection of this specific behavior across the cluster using syscall-level visibility, without modifying application code. Which solution BEST meets this requirement?

A) Enable Kubernetes audit logging at the RequestResponse level for all pods
B) Deploy Falco with the default ruleset, which includes a rule that alerts on 'Terminal shell in container'
C) Apply a NetworkPolicy that denies all egress traffic from the pod
D) Enable readOnlyRootFilesystem on the pod's securityContext

---

**Q39.** You need to configure a Kubernetes audit policy so that requests to read Secrets are logged in full detail (including request and response bodies) for forensic purposes, while routine 'list pods' requests are logged with metadata only to save storage. Which audit level should be assigned to the Secrets rule?

A) None
B) Metadata
C) RequestResponse
D) Request

---

**Q40.** A pod is deployed with securityContext.readOnlyRootFilesystem: true and no writable emptyDir volumes mounted. A compromised process inside the container attempts to write a malicious binary to /tmp on the container's root filesystem. What happens?

A) The kubelet detects the write attempt and immediately kills the pod
B) The write fails with a 'Read-only file system' error at the OS level, since no writable path exists for /tmp
C) The write silently succeeds because readOnlyRootFilesystem only restricts kubectl cp operations
D) The admission controller blocks the pod from starting due to the write attempt

---

**Q41.** A platform team wants to detect anomalous behavior such as unexpected outbound connections, privilege escalation attempts, and sensitive file reads across all running containers, using kernel-level syscall data with minimal performance overhead and without requiring a custom kernel module to be compiled on every node. Which tool and driver combination BEST fits this requirement?

A) Trivy with its built-in runtime scanning mode
B) kube-bench running as a scheduled CronJob
C) OPA Gatekeeper with custom admission webhooks
D) Falco using the eBPF probe driver instead of the kernel module driver

---

**Q42.** You need to enable Kubernetes API server audit logging on a kubeadm-managed cluster, writing structured JSON audit events to a local file based on a custom policy file. Which pair of kube-apiserver flags is REQUIRED to accomplish this?

A) --enable-audit-log and --audit-format=json
B) --audit-webhook-config-file and --audit-log-maxsize
C) --audit-log-path and --audit-policy-file
D) --audit-log-format and --audit-dynamic-configuration

---

**Q43.** When deploying Falco on a managed Kubernetes service where you cannot load custom kernel modules on worker nodes, which statement about Falco's driver options is CORRECT?

A) Falco cannot function at all without the kernel module driver, so it is unusable on such clusters
B) Falco requires SELinux to be disabled cluster-wide before any driver can be loaded
C) Falco supports a modern eBPF probe driver as an alternative to the kernel module, avoiding the need to compile and load kernel modules on the host
D) Falco can use its userspace instrumentation via ptrace instead of syscall monitoring, bypassing driver requirements entirely

---

**Q44.** A production container is compromised and the attacker spawns an interactive shell to explore the filesystem — this could happen via kubectl exec, a reverse shell payload, or a hijacked process, so the entry vector is unpredictable. Which detection mechanism catches this behavior in real time REGARDLESS of how the shell was spawned?

A) Kubernetes audit logging configured at the Metadata level for the pods/exec sub-resource
B) A PodSecurityPolicy denying allowPrivilegeEscalation on the pod spec
C) The default Falco rule 'Terminal shell in container', which alerts on any process matching a shell binary spawned with a controlling tty inside a container, based on syscall-level monitoring
D) The default seccomp profile (RuntimeDefault) applied to the pod

---

**Q45.** You must capture full request AND response bodies in the Kubernetes audit log specifically for pods/exec and pods/attach calls (to reconstruct attacker commands), while keeping audit volume manageable by logging all other resources only at the Metadata level. How should the audit policy be structured to achieve this?

A) Place a rule matching resource 'pods' with resources[].subresources: ['exec','attach'] and level: RequestResponse BEFORE a catch-all rule with level: Metadata
B) Set a single top-level rule with level: RequestResponse that applies to all resources in the cluster
C) Place a rule matching subresources 'exec' and 'attach' with level: Metadata before a catch-all rule with level: Request
D) Add omitStages: ['ResponseStarted'] to the audit policy so all matching rules automatically capture full response bodies

---

**Q46.** You want a Falco rule that detects a container process attempting to write into a path that is expected to be read-only in an immutable container image (e.g., under /etc or a binary directory), signaling a possible runtime tampering attempt. Which combination of Falco fields does the default falco_rules.yaml use to build rules like 'Write below etc' for this purpose?

A) evt.type=execve combined with proc.name in (bash, sh, dash)
B) A Kubernetes audit rule matching verb=update on the pod's spec.containers[].volumeMounts field
C) evt.type in (open, openat) and evt.is_open_write=true, combined with fd.name matched against a list of protected read-only paths
D) A seccomp profile that unconditionally blocks the write() syscall for the pod

---

