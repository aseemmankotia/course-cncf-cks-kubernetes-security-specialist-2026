## Chapter 11: Monitoring II: Audit Logging & Immutable Containers — Practice Questions

**Q1.** A security team wants the API server to log full request AND response bodies only for 'secrets' resources, while all other resources should only log metadata. Which audit policy structure achieves this?

A) A single rule with level: RequestResponse and no resource restriction
B) Two rules: a rule matching group '' resources ['secrets'] with level RequestResponse placed BEFORE a catch-all rule with level Metadata
C) Two rules: a catch-all rule with level Metadata placed BEFORE a rule matching resources ['secrets'] with level RequestResponse
D) A single rule with level: Request and omitStages for secrets

<details>
<summary>Answer</summary>

**Correct: B**

Audit policy rules are evaluated top-down and the FIRST matching rule wins, so the more specific 'secrets' rule with RequestResponse must come before the generic catch-all Metadata rule.

**Why the others are wrong:**
- This would log RequestResponse for every resource, exposing full bodies of all objects including large or sensitive non-secret data unnecessarily.
- Since the catch-all Metadata rule comes first, it matches secrets requests too, so the more specific RequestResponse rule for secrets never triggers.
- 'Request' level only logs the request body, not the response body, and omitStages is unrelated to per-resource level selection.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** You configure the API server audit log backend with --audit-log-path=/var/log/kubernetes/audit.log but no rotation flags are set. What is the risk?

A) The audit log will stop after 24 hours
B) Kubernetes automatically rotates logs every 100MB by default
C) The API server will fail to start without rotation flags
D) The audit log file will grow unbounded and can fill disk, since maxage/maxbackup/maxsize default only apply when explicitly set

<details>
<summary>Answer</summary>

**Correct: D**

Without --audit-log-maxsize, --audit-log-maxage, and --audit-log-maxbackup, the log backend does not rotate or limit the file, risking disk exhaustion.

**Why the others are wrong:**
- There is no time-based cutoff unrelated to maxage; without maxage the file simply keeps growing.
- Kubernetes has no built-in default rotation size; rotation only occurs when maxsize is explicitly configured.
- The API server starts fine with just audit-log-path set; rotation flags are optional but strongly recommended.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A pod runs a container with securityContext.readOnlyRootFilesystem: true. At runtime, the application tries to write a temporary file to /tmp and the write fails. What is the BEST fix that preserves the immutable root filesystem?

A) Mount an emptyDir volume at /tmp
B) Set readOnlyRootFilesystem to false
C) Add allowPrivilegeEscalation: true to the container
D) Run the container as root so it can write anywhere

<details>
<summary>Answer</summary>

**Correct: A**

Mounting a writable emptyDir (or other volume) at specific paths like /tmp lets the app write where needed while the rest of the root filesystem remains immutable, satisfying the exam's expected hardening pattern.

**Why the others are wrong:**
- Disabling readOnlyRootFilesystem defeats the security control the exam is testing you to preserve.
- allowPrivilegeEscalation controls privilege escalation via setuid binaries, unrelated to filesystem writability.
- Running as root does not bypass a read-only root filesystem restriction; root is still blocked from writing to a read-only mount.

</details>

---

**Q4.** Which audit policy level records the request metadata (user, timestamp, resource, verb) but NOT the request or response body?

A) None
B) Request
C) RequestResponse
D) Metadata

<details>
<summary>Answer</summary>

**Correct: D**

Metadata level logs event metadata only, excluding request and response bodies — the lowest verbosity level that still logs an event.

**Why the others are wrong:**
- None means the event matching this rule is not logged at all.
- Request logs metadata plus the request body but not the response body.
- RequestResponse logs metadata, request body, and response body — the highest verbosity level.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** During an incident, you see a Falco alert 'Set Setuid or Setgid bit' fired against a running pod. Which correlated audit log entry would BEST help confirm whether this was an authorized deployment action versus a runtime attack?

A) The kube-scheduler leader election log around the same timestamp
B) The kubelet's cAdvisor metrics for CPU usage on the node
C) The etcd defragmentation logs from the same day
D) The kube-apiserver audit log entry showing which user/service account issued the exec or created the pod spec with the setuid capability, correlated by timestamp and pod UID

<details>
<summary>Answer</summary>

**Correct: D**

Correlating the Falco runtime alert's timestamp/pod with the API server audit log lets you trace which identity created/modified the pod or executed into it, distinguishing legitimate CI/CD actions from an attacker's exec session.

**Why the others are wrong:**
- Scheduler leader election logs are unrelated to workload identity or pod actions.
- CPU metrics don't reveal who performed an action or authorization context.
- etcd defragmentation is a maintenance operation unrelated to security incident attribution.

</details>

---

**Q6.** You need audit events sent to an external SIEM in near real-time in addition to a local log file. Which combination of API server flags accomplishes this?

A) --audit-log-path and --audit-log-format only
B) --audit-dynamic-configuration alone
C) --audit-log-path (for the local file) and --audit-webhook-config-file (for the external SIEM webhook)
D) --enable-admission-plugins=EventRateLimit alone

<details>
<summary>Answer</summary>

**Correct: C**

The log backend (--audit-log-path) and webhook backend (--audit-webhook-config-file) can run simultaneously, each governed by its own audit policy, allowing local file logging plus webhook forwarding to a SIEM.

**Why the others are wrong:**
- --audit-log-format only controls format (legacy-text vs json) of the existing log backend, it does not add SIEM forwarding.
- --audit-dynamic-configuration was a deprecated/removed alpha feature for dynamically changing audit config, not a delivery mechanism to external systems.
- EventRateLimit is an admission controller that throttles requests; it has nothing to do with audit log delivery.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which file permission setting is the recommended hardening practice for the Kubernetes API server audit log file on disk?

A) 600 (or more restrictive), owned by root, so only the API server process can read/write it
B) 644, owned by any user
C) 777, so all system users can review logs
D) No permission restriction is needed since audit logs are non-sensitive

<details>
<summary>Answer</summary>

**Correct: A**

Audit logs can contain sensitive request/response data (e.g. Secret payloads at RequestResponse level) and must be restricted to root/API-server only, typically 600 permissions, to prevent tampering or unauthorized reads.

**Why the others are wrong:**
- 644 allows any local user to read the file, potentially exposing sensitive audit data.
- 777 grants world write access, allowing tampering or deletion of audit evidence — a CIS Benchmark failure.
- Audit logs frequently contain secrets, tokens, and user identity data and must be protected, not left unrestricted.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A container has securityContext.allowPrivilegeEscalation: false and readOnlyRootFilesystem: true but is still running as root (runAsUser: 0). Which statement correctly describes the residual risk?

A) There is no residual risk; these two settings alone make the container fully non-privileged
B) The container still runs as root, so a kernel or capability-based exploit could still leverage root privileges inside the container even though it cannot escalate further or persist writes to the root filesystem
C) readOnlyRootFilesystem automatically forces runAsNonRoot behavior
D) allowPrivilegeEscalation: false prevents the process from ever running as UID 0

<details>
<summary>Answer</summary>

**Correct: B**

allowPrivilegeEscalation and readOnlyRootFilesystem reduce specific attack surfaces (setuid escalation and persistent writes) but do not prevent the process from running as root; runAsNonRoot must be set separately to eliminate root-level risk.

**Why the others are wrong:**
- These controls address escalation and filesystem writes specifically — they do not eliminate risk from the process already running with root UID/capabilities.
- readOnlyRootFilesystem has no relationship to the runAsUser/runAsNonRoot settings; they are independent fields.
- allowPrivilegeEscalation only blocks gaining MORE privileges than the parent process (e.g. via setuid binaries); it does not change the starting UID of the process.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

