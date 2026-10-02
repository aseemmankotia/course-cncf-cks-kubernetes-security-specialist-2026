## Chapter 10: Monitoring I: Falco Runtime Detection — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Falco syscall capture mechanisms (3 options)? | Kernel module (kmod), legacy eBPF probe, modern eBPF (CO-RE) — modern eBPF is default/recommended, no kernel-specific compile needed. Exam tip: know all three exist and eBPF avoids per-kernel module builds. |
| 2 | Required fields in a Falco custom rule? | rule, desc, condition, output, priority. Optional: tags. Exam tip: missing 'condition' or 'output' makes the rule invalid. |
| 3 | Falco priority levels, highest to lowest (8 total)? | Emergency, Alert, Critical, Error, Warning, Notice, Informational, Debug. Exam tip: default shell-in-container rule fires at WARNING by default. |
| 4 | Falco vs AppArmor — detection or prevention? | Falco = detection (observes syscalls, alerts, does not block). AppArmor = prevention (static MAC profile blocks disallowed syscalls before execution). Exam tip: this pairing is a classic 'detect vs prevent' distinction question. |
| 5 | Falco condition syntax for path prefix match? | fd.name startswith /etc — Exam tip: 'startswith', 'contains', 'icontains', 'in', 'exists' are valid Falco condition operators. |
| 6 | How to detect a shell spawned inside a container (core logic)? | condition: spawned_process and container.id != host and proc.name in (shell_binaries) — Exam tip: container.id != host is the key filter excluding host-level shells. |
| 7 | Falco output channels available? | stdout, file, syslog, program (pipe to external command), HTTP/HTTPS webhook, gRPC (structured streaming API). Exam tip: gRPC is used by falcosidekick for multi-destination fan-out. |
| 8 | How to add custom rules without editing the shipped default rules file? | Place custom rules in a separate file (e.g. a rules directory) and load/append it via falco.yaml config. Exam tip: never hand-edit the default falco_rules.yaml — upgrades will overwrite it. |
| 9 | Falco macro vs list — difference? | A macro is a reusable named condition snippet (e.g. spawned_process); a list is a reusable named array of values (e.g. shell_binaries). Exam tip: macros and lists reduce duplication across custom rules. |
| 10 | Command to validate a Falco rules file before deploying? | falco --validate <rules_file.yaml> — Exam tip: always validate custom rules to catch syntax errors before rolling out to a running cluster. |
| 11 | What does Falco NOT do that a prevention tool would? | Falco does not block/kill the offending process or syscall itself — it only generates an alert/event. Blocking requires external response automation (e.g. Falco Talon, falcosidekick response actions) or a separate prevention control. |
| 12 | Key output template fields for triaging a shell-in-container alert? | %user.name (who), %container.name (where), %proc.name (what ran), %proc.pname (parent — legit exec vs injection), %proc.cmdline (full command). Exam tip: %proc.pname is critical for distinguishing kubectl exec from a compromise. |
