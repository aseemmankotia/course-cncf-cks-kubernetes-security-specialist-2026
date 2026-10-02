## Chapter 5: Microservice Security I: Pod Security Admission & Policy Engines — Practice Questions

**Q1.** A cluster admin must ensure that all pods in the 'prod' namespace are blocked from running as privileged or using hostPath volumes, while pods in 'staging' should only generate warnings for the same violations but still be admitted. Which pair of namespace label sets achieves this?

A) prod: pod-security.kubernetes.io/audit=restricted; staging: pod-security.kubernetes.io/enforce=baseline
B) prod: pod-security.kubernetes.io/enforce=restricted; staging: pod-security.kubernetes.io/warn=restricted
C) prod: pod-security.kubernetes.io/enforce=privileged; staging: pod-security.kubernetes.io/warn=privileged
D) prod: podsecuritypolicy.kubernetes.io/enforce=restricted; staging: podsecuritypolicy.kubernetes.io/warn=restricted

<details>
<summary>Answer</summary>

**Correct: B**

The 'enforce' mode label actually blocks non-compliant pods, and 'restricted' is the level that disallows privileged containers and hostPath. The 'warn' mode only surfaces a client-side warning without blocking admission.

**Why the others are wrong:**
- Audit only writes to the audit log — it does not block pods, so prod would not be protected.
- Privileged level allows everything, including privileged containers and hostPath, so this fails the requirement entirely.
- PodSecurityPolicy (PSP) is a removed API object (gone since v1.25) — the label prefix 'podsecuritypolicy.kubernetes.io' is not valid for PSA.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team wants to add a policy that mutates every incoming Pod to inject a default resource limit, validates that images come only from an approved registry, and generates a companion NetworkPolicy for each new namespace — all without writing Rego. Which tool should they use?

A) OPA Gatekeeper with a single ConstraintTemplate
B) PodSecurityPolicy with a custom admission webhook
C) Pod Security Admission with the restricted profile
D) Kyverno with ClusterPolicy validate, mutate, and generate rules

<details>
<summary>Answer</summary>

**Correct: D**

Kyverno is a Kubernetes-native policy engine that expresses validate, mutate, and generate rules declaratively in YAML (no Rego required), matching all three requirements in one ClusterPolicy.

**Why the others are wrong:**
- Gatekeeper requires writing Rego inside the ConstraintTemplate and has no native mutate or generate rule types — it is validation-focused.
- PodSecurityPolicy is deprecated and removed as of Kubernetes v1.25; it cannot be used at all.
- PSA only enforces a fixed set of Pod Security Standards levels — it cannot mutate pods or generate other resources.

</details>

---

**Q3.** Which namespace label configuration causes the API server to log a warning to kubectl output for baseline violations but still admit the pod?

A) pod-security.kubernetes.io/enforce=baseline
B) pod-security.kubernetes.io/audit=baseline
C) pod-security.kubernetes.io/warn=baseline
D) pod-security.kubernetes.io/warn-version=baseline

<details>
<summary>Answer</summary>

**Correct: C**

The 'warn' mode label returns a user-facing warning via the API response (visible in kubectl) for violations of the specified level, without blocking the pod.

**Why the others are wrong:**
- Enforce mode blocks the pod outright instead of just warning.
- Audit mode writes to the audit log, not to the user's kubectl output.
- '-version' suffix labels pin the PSS version to check against (e.g. latest or v1.30); they don't set the mode itself and 'baseline' is not a valid version value.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** You wrote a new Kyverno ClusterPolicy to block containers running as root. Before setting validationFailureAction to Enforce cluster-wide, what is the recommended way to safely test its impact?

A) Delete all existing pods and observe which ones fail to reschedule
B) Apply the policy only in the kube-system namespace first
C) Set validationFailureAction to Audit and review generated PolicyReport violations before switching to Enforce
D) Use kubectl auth can-i to simulate the policy decision

<details>
<summary>Answer</summary>

**Correct: C**

Audit mode admits non-compliant resources but records violations in PolicyReport objects, letting teams assess blast radius before flipping to Enforce.

**Why the others are wrong:**
- Deleting live pods to test policy impact is destructive and not how dry-run testing is done.
- kube-system often contains privileged system pods that would break the cluster if policy failed — it's the worst namespace to test against.
- kubectl auth can-i tests RBAC permissions, not policy engine validation rules.

</details>

---

**Q5.** In OPA Gatekeeper, what is the relationship between a ConstraintTemplate and a Constraint?

A) A Constraint defines the Rego logic and schema; the ConstraintTemplate is an instance that applies it with specific parameters
B) They are interchangeable terms for the same CRD object
C) A ConstraintTemplate only works for mutating admission, while Constraint only works for validating admission
D) A ConstraintTemplate defines the Rego logic and generates a new CRD kind; a Constraint is an instance of that CRD specifying match criteria and parameters

<details>
<summary>Answer</summary>

**Correct: D**

ConstraintTemplate defines the Rego policy logic plus an OpenAPI schema for parameters, and Gatekeeper generates a new CRD from it. You then create Constraint resources of that generated kind to actually enforce the policy with specific match/parameter values.

**Why the others are wrong:**
- This reverses the roles — the template holds logic, the constraint is the instance, not vice versa.
- They are distinct object kinds serving different purposes; a Constraint cannot exist without its corresponding ConstraintTemplate.
- Gatekeeper is primarily a validating admission controller; native mutation support was added later via Mutation CRDs, unrelated to this template/constraint split.

</details>

---

**Q6.** A cluster was upgraded from Kubernetes v1.24 to v1.25. Pods that previously relied on PodSecurityPolicy for restricting privilege escalation are now failing to schedule with an unrecognized API error. What is the correct remediation?

A) Recreate the PodSecurityPolicy objects using the v1beta1 API version
B) Enable the PodSecurityPolicy admission controller flag on the API server
C) Replace PodSecurityPolicy enforcement with Pod Security Admission namespace labels and/or an equivalent Kyverno/Gatekeeper policy
D) Downgrade the cluster back to v1.24 to restore PSP support

<details>
<summary>Answer</summary>

**Correct: C**

PodSecurityPolicy was removed entirely in Kubernetes v1.25 (deprecated since v1.21). Its replacement is Pod Security Admission for baseline enforcement, supplemented by policy engines like Kyverno or Gatekeeper for anything PSA doesn't cover.

**Why the others are wrong:**
- The PSP API (policy/v1beta1) no longer exists in v1.25+ clusters at all — recreating it will fail.
- There is no PSP admission controller flag to re-enable in v1.25+; the entire API and controller were removed, not just disabled.
- Downgrading is not a real remediation and reintroduces an already-deprecated, insecure mechanism instead of adopting the supported replacement.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which Pod Security Standard level permits privileged containers, host namespaces (hostNetwork, hostPID, hostIPC), and arbitrary hostPath volumes?

A) privileged
B) restricted
C) baseline
D) audit

<details>
<summary>Answer</summary>

**Correct: A**

The 'privileged' PSS level is fully unrestricted by design — it exists to allow trusted, cluster-critical workloads (e.g. CNI plugins, node agents) that need broad host access.

**Why the others are wrong:**
- restricted is the most locked-down level, disallowing privilege escalation, host namespaces, and requiring non-root, seccomp, and dropped capabilities.
- baseline blocks known privilege escalations (e.g. no privileged containers, no host namespaces) but is more permissive than restricted.
- 'audit' is a PSA enforcement mode, not a security level — it cannot be confused with a level in a well-formed answer, but is a common distractor.

</details>

---

**Q8.** You need to write a Kyverno rule that blocks any Pod from mounting the host's Docker socket (/var/run/docker.sock) as a volume. Which rule type should you use?

A) A generate rule that creates a NetworkPolicy
B) A mutate rule that overwrites the volume path
C) A validate rule with a pattern that denies matching hostPath volumes
D) A verifyImages rule checking image signatures

<details>
<summary>Answer</summary>

**Correct: C**

Blocking a specific dangerous hostPath value at admission time is a validation concern — Kyverno's validate rule with a deny/pattern match against spec.volumes[].hostPath.path is the correct construct.

**Why the others are wrong:**
- generate rules create auxiliary resources (like NetworkPolicies or ConfigMaps); they don't block or check existing pod fields.
- mutate rules silently change the resource rather than rejecting it — an attacker's pod would still run, just altered, which is not the goal here.
- verifyImages is specifically for validating image signatures/provenance (e.g. cosign), unrelated to hostPath volumes.

</details>

---

