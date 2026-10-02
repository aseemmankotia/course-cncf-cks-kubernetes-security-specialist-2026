## Chapter 11 Cheat Sheet: Audit Logging & Immutable Containers

### Audit Policy Levels (least → most verbose)
| Level | Logs |
|---|---|
| None | Nothing (drop event) |
| Metadata | User, timestamp, resource, verb — no bodies |
| Request | Metadata + request body |
| RequestResponse | Metadata + request body + response body |

**Rule matching:** top-down, first match wins. Specific rules go BEFORE catch-all rules.

### API Server Audit Flags
| Flag | Purpose |
|---|---|
| `--audit-policy-file` | Path to policy YAML (required to enable auditing) |
| `--audit-log-path` | File path for log backend (`-` = stdout) |
| `--audit-log-maxage` | Days to retain rotated logs |
| `--audit-log-maxbackup` | Number of rotated files to keep |
| `--audit-log-maxsize` | MB before rotation triggers |
| `--audit-log-format` | `json` (default) or `legacy-text` |
| `--audit-webhook-config-file` | Kubeconfig-style file pointing to external SIEM/webhook |
| `--audit-webhook-batch-max-size` | Max events per webhook batch |

### Sample Minimal Audit Policy
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
- level: Metadata
```

### readOnlyRootFilesystem Pattern
```yaml
securityContext:
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  runAsNonRoot: true
volumeMounts:
- name: tmp
  mountPath: /tmp
volumes:
- name: tmp
  emptyDir: {}
```
**Gotcha:** `readOnlyRootFilesystem` ≠ non-root. Must combine with `runAsNonRoot`/`runAsUser` for full hardening.

### Security Context Field Comparison
| Field | Blocks |
|---|---|
| `readOnlyRootFilesystem` | Writes to root FS (not mounted volumes) |
| `allowPrivilegeEscalation` | Gaining MORE privileges than parent process |
| `runAsNonRoot` | Process starting as UID 0 |
| `privileged` | Full host device/kernel access |

### Audit Log File Protection
- Permissions: `600`, owner `root:root`
- Never world-readable/writable (CIS Benchmark check)
- Store on a dedicated partition to avoid disk exhaustion from other processes

### Falco ↔ Audit Log Correlation Workflow
1. Falco alert fires (e.g. `Set Setuid or Setgid bit`, `Terminal shell in container`)
2. Extract timestamp + pod/container UID from alert
3. Search kube-apiserver audit log for matching pod UID around that timestamp
4. Identify `user.username` / service account and the exact verb (create/exec/patch)
5. Determine if action was authorized (CI/CD SA) or anomalous (unexpected user/exec)

### Common Exam Traps
- Forgetting `--audit-policy-file` is mandatory to activate ANY audit backend
- Assuming default rotation exists — it doesn't, must set maxage/maxbackup/maxsize explicitly
- Believing `allowPrivilegeEscalation: false` blocks root execution — it does not
- Placing catch-all `Metadata` rule before specific `RequestResponse` rule (wrong order = specific rule never triggers)
