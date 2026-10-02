## Chapter 11: Monitoring II: Audit Logging & Immutable Containers — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | What are the 4 Kubernetes audit policy levels, from least to most verbose? | None → Metadata → Request → RequestResponse. Exam tip: Metadata logs no bodies; RequestResponse logs both request and response bodies (can be huge for Secrets/ConfigMaps). |
| 2 | Flag to specify the audit policy file for kube-apiserver? | --audit-policy-file=/etc/kubernetes/audit-policy.yaml. Exam tip: must be paired with --audit-log-path (or webhook config) to actually write events. |
| 3 | Flag to set the audit log output file path? | --audit-log-path=/var/log/kubernetes/audit/audit.log. Setting it to '-' sends audit logs to stdout instead of a file. |
| 4 | Flags controlling audit log rotation on the log backend? | --audit-log-maxage (days), --audit-log-maxbackup (# of old files), --audit-log-maxsize (MB before rotation). Exam tip: without these, the log file grows unbounded. |
| 5 | How does audit policy rule matching work when multiple rules could apply to an event? | First matching rule (top-down order) wins. Exam tip: put more specific rules BEFORE broader catch-all rules in the policy file. |
| 6 | securityContext field to make a container's root filesystem immutable at runtime? | readOnlyRootFilesystem: true. Exam tip: apps needing to write to /tmp or /var/cache require an explicit writable volume (e.g. emptyDir) mounted at that path. |
| 7 | Does allowPrivilegeEscalation: false prevent a container from running as root? | No. It only blocks processes from gaining MORE privileges than their parent (e.g. via setuid binaries). Combine with runAsNonRoot: true to actually block root execution. |
| 8 | Flag to configure a webhook backend for forwarding audit events to an external SIEM? | --audit-webhook-config-file=<kubeconfig-style file with SIEM endpoint>. Can run simultaneously with the log file backend, each governed by its own --audit-policy-file reference. |
| 9 | Recommended file permissions for the audit log file on the control plane node? | 600 (root:root). Exam tip: CIS Benchmark checks flag world-readable/writable audit logs as findings since they may contain sensitive request bodies. |
| 10 | How do you correlate a Falco runtime alert with the source of a suspicious action? | Match the Falco alert's timestamp + pod UID/container ID against kube-apiserver audit log entries to identify the user/service account and exact API call that created or modified the workload. |
| 11 | Which Falco rule commonly fires on privilege escalation attempts inside a container? | 'Set Setuid or Setgid bit' and 'Terminal shell in container' are classic Falco default rules used to detect privilege escalation and unauthorized interactive access. Exam tip: know these rule NAMES appear in falco output. |
| 12 | What audit policy 'omitStages' field does? | Excludes specific request stages (e.g. RequestReceived) from being logged for a rule, reducing noise from duplicate stage entries per request. Exam tip: often used to skip the 'RequestReceived' stage since it's redundant with 'ResponseComplete'. |
