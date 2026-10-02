## Chapter 5: Microservice Security I: Pod Security Admission & Policy Engines — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | What are the three Pod Security Standard (PSS) levels used by Pod Security Admission? | privileged, baseline, restricted (from least to most restrictive). Exam tip: 'restricted' is the hardened production default; 'privileged' is unrestricted, used for system components. |
| 2 | What are the three PSA namespace label modes? | enforce, audit, warn. Label format: pod-security.kubernetes.io/<mode>=<level>. Exam tip: only 'enforce' actually blocks pod admission. |
| 3 | What Kubernetes version removed PodSecurityPolicy (PSP) entirely? | v1.25. Deprecated since v1.21. Exam tip: PSP replacement path is Pod Security Admission plus optional Kyverno/Gatekeeper for gaps. |
| 4 | Kyverno vs OPA Gatekeeper: key syntax difference? | Kyverno policies are written in native Kubernetes YAML (validate/mutate/generate rules). Gatekeeper ConstraintTemplates embed Rego policy logic and generate a CRD kind that Constraints instantiate. |
| 5 | What does a Gatekeeper ConstraintTemplate define? | The Rego policy logic plus an OpenAPI schema for parameters. Applying it generates a new CRD kind. Exam tip: you still need a separate Constraint object of that generated kind to enforce anything. |
| 6 | What does a Gatekeeper Constraint define? | An instance of a ConstraintTemplate-generated CRD, specifying 'match' (which resources/namespaces to target) and 'parameters' (policy-specific values). |
| 7 | Kyverno validationFailureAction: what do Enforce and Audit do? | Enforce blocks non-compliant resources at admission. Audit admits them but records violations (e.g. in a PolicyReport) for review. Exam tip: use Audit first to test blast radius before switching to Enforce. |
| 8 | Which PSS level disallows privileged containers and host namespaces but is more permissive than 'restricted'? | baseline — blocks known privilege escalations (privileged containers, hostNetwork/PID/IPC, most hostPath) but does not require non-root or seccomp like restricted does. |
| 9 | What label sets the Pod Security Standard version PSA checks against for a given mode? | pod-security.kubernetes.io/<mode>-version=<latest\|vX.Y>, e.g. pod-security.kubernetes.io/enforce-version=latest. Optional — defaults to latest if omitted. |
| 10 | Kyverno rule types available in a ClusterPolicy? | validate, mutate, generate, and verifyImages. Exam tip: verifyImages is used for image signature/provenance checks (e.g. with cosign), not general field validation. |
| 11 | How do you safely test a new admission policy without blocking real workloads? | Use Audit mode (Kyverno validationFailureAction=Audit, or Gatekeeper 'dryrun' enforcementAction) and inspect generated violation reports before switching to Enforce/deny. |
| 12 | PSA restricted level: which securityContext fields are required? | runAsNonRoot: true, allowPrivilegeEscalation: false, seccompProfile.type: RuntimeDefault (or Localhost), and capabilities must drop ALL (only NET_BIND_SERVICE may optionally be added). |
