## Chapter 8: Supply Chain Security I: Image Footprint & Vulnerability Scanning — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Trivy command to fail a CI build only on CRITICAL vulnerabilities? | trivy image --severity CRITICAL --exit-code 1 <image>. Exam tip: default exit-code is 0, which never fails the pipeline even with CVEs present. |
| 2 | CVSS score range for CRITICAL severity? | 9.0–10.0. HIGH = 7.0–8.9, MEDIUM = 4.0–6.9, LOW = 0.1–3.9. Exam tip: remediation priority order is always CRITICAL > HIGH. |
| 3 | What does a distroless image NOT contain? | No shell, no package manager, no OS utilities. Exam tip: an attacker who compromises the container can't run `sh` or install additional tools. |
| 4 | Common distroless base image family? | gcr.io/distroless/{static,base,cc,java,python3}. Exam tip: 'static' variant has no libc — use it for statically-linked Go binaries only. |
| 5 | Multi-stage Dockerfile syntax to copy an artifact from an earlier stage? | COPY --from=<stage-name-or-index> <src> <dest>. Exam tip: the final FROM must be a minimal/distroless base — copying into a full OS image defeats the purpose. |
| 6 | Two kube-apiserver flags required to enable ImagePolicyWebhook? | --enable-admission-plugins=ImagePolicyWebhook and --admission-control-config-file=<path>. Exam tip: config file sets kubeConfigFile, defaultAllow, allowTTL, denyTTL. |
| 7 | Kyverno resource kinds for admission policies? | ClusterPolicy (cluster-scoped) or Policy (namespaced). Exam tip: validate.pattern rules restrict spec.containers[].image to an approved registry prefix. |
| 8 | Gatekeeper's two core components to enforce policy? | ConstraintTemplate (Rego logic) + Constraint (applies the template with parameters). Exam tip: Gatekeeper is OPA-based; Kyverno uses native YAML with no Rego required. |
| 9 | Trivy subcommand to scan a running Kubernetes cluster's workloads? | trivy k8s --report summary cluster (or via trivy-operator). Exam tip: distinct from `trivy image`, which scans a single image reference, not live cluster resources. |
| 10 | How does Kyverno block images from untrusted registries? | validate.pattern rule matching spec.containers[].image against 'approvedregistry.io/*' with validationFailureAction: Enforce. Exam tip: 'Audit' mode logs violations but doesn't block — exam distinguishes Enforce vs Audit. |
| 11 | Why prefer alpine over ubuntu for footprint reduction, but distroless over alpine for security? | Alpine (~5MB) has far fewer packages/CVEs than ubuntu (~70MB+), but still ships apk and a shell. Distroless removes both entirely for the smallest attack surface. Exam tip: alpine is a middle ground, not the most secure option. |
| 12 | Default severity behavior of `trivy image` when --severity is omitted? | Reports ALL severities (UNKNOWN, LOW, MEDIUM, HIGH, CRITICAL). Exam tip: for CI gating you must explicitly pass --severity HIGH,CRITICAL to focus the exit-code check. |
