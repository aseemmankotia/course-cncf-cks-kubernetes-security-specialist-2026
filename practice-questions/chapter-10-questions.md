## Chapter 10: Monitoring I: Falco Runtime Detection — Practice Questions

**Q1.** A security team wants Falco to alert whenever an interactive shell is spawned inside any running container, but Falco should not block the shell from executing. Which statement correctly describes this capability?

A) Falco will block the shell process automatically using its built-in seccomp enforcement
B) Falco requires a Kubernetes admission webhook to detect shell spawns in containers
C) Falco can only detect shell spawns if the container runs as a privileged pod
D) Falco will detect and log the event, but cannot prevent the shell from executing because it is a detection engine

<details>
<summary>Answer</summary>

**Correct: D**

Falco observes syscalls in real time via eBPF/kernel module and generates alerts; it has no native enforcement/blocking mechanism — response actions require external tooling.

**Why the others are wrong:**
- Falco has no built-in seccomp enforcement; seccomp is a separate kernel prevention mechanism configured independently
- Admission webhooks operate at API server request time, not at runtime syscall level — irrelevant to shell detection inside a running container
- Falco detects syscalls regardless of privilege level; privileged mode is not a requirement for detection

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A custom Falco rule must fire only when a process writes to a file under /etc inside a container, excluding writes performed by trusted config-management binaries. Which condition syntax correctly expresses this?

A) condition: evt.type=open or fd.name=/etc and proc.name != trusted_binaries
B) condition: evt.type=open and fd.name startswith /etc and not proc.name in (trusted_binaries)
C) condition: fd.directory == /etc && process.write == true
D) condition: container.id exists and file.path=/etc

<details>
<summary>Answer</summary>

**Correct: B**

Falco rule language uses evt.type, fd.name with startswith for path prefix matching, and 'not ... in (list)' to exclude a set of trusted process names — this is valid, real Falco syntax.

**Why the others are wrong:**
- Uses 'or' instead of 'and', which changes rule logic to match ANY open OR any /etc path, and compares proc.name to a list without 'in', which is invalid syntax
- 'fd.directory' and 'process.write' and '&&' are not valid Falco filter fields/operators
- 'file.path' is not a valid Falco field; the correct field is fd.name, and this condition doesn't check for a write operation

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An organization needs Falco alerts to be consumed by falcosidekick for routing to Slack, Prometheus, and a Kubernetes response engine simultaneously. Which Falco output channel is designed for this streaming integration pattern?

A) gRPC output consumed by an external client
B) syslog output only
C) file output written to /var/log/falco.log
D) stdout piped directly to journald

<details>
<summary>Answer</summary>

**Correct: A**

Falco's gRPC output/API (grpc_output enabled in falco.yaml) exposes a structured, subscribable event stream that tools like falcosidekick consume to fan out alerts to multiple downstream destinations.

**Why the others are wrong:**
- Syslog is a valid output but is a single flat text stream not designed for structured multi-destination fan-out by external clients
- File output requires a separate log-tailing agent and lacks the structured streaming API gRPC provides
- stdout requires container log scraping and lacks the direct client-subscribe model gRPC provides

</details>

---

**Q4.** Which BEST describes the fundamental difference between Falco and a static policy enforcement tool like AppArmor for a CKS exam scenario?

A) Falco enforces mandatory access control profiles while AppArmor only logs violations
B) Falco and AppArmor are functionally identical and interchangeable for container hardening
C) Falco performs behavioral, runtime detection of anomalous syscall patterns, while AppArmor statically restricts allowed syscalls/paths before execution occurs
D) AppArmor requires eBPF while Falco requires a kernel module

<details>
<summary>Answer</summary>

**Correct: C**

Falco is a detection-only runtime security tool observing behavior against rules; AppArmor is a static, preventive MAC profile enforced by the kernel that blocks disallowed actions before they occur.

**Why the others are wrong:**
- This reverses the roles — AppArmor enforces, Falco logs/alerts, not the other way around
- They serve different purposes (detection vs prevention) and are commonly used together, not interchangeably
- eBPF/kernel module choice relates to Falco's syscall capture method, not a distinguishing factor versus AppArmor

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A cluster administrator wants to add a new custom Falco rule without modifying the default shipped rules file. Which approach follows Falco best practice?

A) Place the custom rule in a separate file (e.g. under the rules directory) and reference it via the rules_file/append configuration in falco.yaml
B) Directly edit falco_rules.yaml to insert the custom rule at the top
C) Restart the falco.service with a --custom-rule flag pointing to inline YAML text
D) Custom rules cannot be added; only default rules are supported

<details>
<summary>Answer</summary>

**Correct: A**

Best practice is to keep custom rules in a separate file (e.g. a rules.d directory or a dedicated custom rules yaml) and have falco.yaml load/append it, preserving upgradeability of the default rules file.

**Why the others are wrong:**
- Editing the shipped default rules file directly makes upgrades overwrite custom logic and is explicitly discouraged
- There is no such --custom-rule inline flag in Falco's CLI for defining rule content
- Falco fully supports and is designed for custom rule authoring — this option is factually false

</details>

---

**Q6.** Falco captures syscall activity to generate runtime alerts. Which mechanism does modern Falco use by default to source these syscall events with the lowest operational overhead and without requiring a custom kernel module build?

A) Legacy kernel module (kmod) driver
B) Modern eBPF driver (CO-RE)
C) strace attached to every container process
D) Kubernetes audit logs

<details>
<summary>Answer</summary>

**Correct: B**

The modern eBPF probe (CO-RE) is Falco's default/recommended syscall source, avoiding the need to compile a kernel module against the host kernel headers.

**Why the others are wrong:**
- The kernel module driver works but requires building/loading a module matched to the exact kernel version — higher operational overhead
- strace is a debugging tool with heavy per-process overhead and is not how Falco sources events
- Kubernetes audit logs capture API server requests, not host syscalls, and are a separate data source entirely

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A default Falco rule fires with priority WARNING when a shell is spawned inside a container. The security team wants this specific condition escalated to CRITICAL without touching the built-in rule definition. What is the correct way to achieve this?

A) Delete the default rule from falco_rules.yaml and hope Falco regenerates it at CRITICAL
B) Change the global default_priority setting in falco.yaml, which applies only to this one rule
C) Priorities cannot be changed after a rule is defined; the rule must be entirely rewritten from scratch every time
D) Use a rule override in a custom rules file with the same rule name and 'override: priority: replace' semantics (or append a new rule for the specific case) with priority set to CRITICAL, loaded after the default file

<details>
<summary>Answer</summary>

**Correct: D**

Falco supports appending/overriding an existing rule by name in a subsequently loaded rules file, allowing fields like priority to be replaced without editing the shipped default file.

**Why the others are wrong:**
- Deleting the rule removes detection entirely and does not regenerate it at a new priority
- There is no global default_priority setting that selectively targets a single rule — priority is per-rule
- Falco explicitly supports rule append/override mechanics; full rewrite is unnecessary and not required

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A Falco alert output template contains: 'Shell spawned in container (user=%user.name container=%container.name shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline)'. What is the purpose of the %proc.pname field in this exam-relevant context?

A) It shows the process ID number of the shell
B) It shows the parent process name, helping distinguish a legitimate exec (e.g. via kubectl exec) from a malicious injection path
C) It shows the container image name pulled from the registry
D) It is a deprecated field with no functional value in current Falco versions

<details>
<summary>Answer</summary>

**Correct: B**

%proc.pname resolves to the parent process name, which is critical context for triage — e.g. a shell parented by 'containerd-shim' via kubectl exec vs one parented by an unexpected process indicating compromise.

**Why the others are wrong:**
- Process ID is provided by %proc.pid, not %proc.pname
- Container image is provided by %container.image.repository/%container.image.tag, not %proc.pname
- %proc.pname is an actively used, standard output field in current Falco rule syntax

</details>

---

