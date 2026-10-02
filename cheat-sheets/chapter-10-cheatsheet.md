## Chapter 10 Cheat Sheet — Falco Runtime Detection

### Falco Architecture
| Component | Role |
|---|---|
| Syscall source | Modern eBPF (CO-RE, default), legacy eBPF probe, or kernel module (kmod) |
| Rules engine | Evaluates syscall/process events against loaded rule conditions |
| Output engine | Formats and dispatches alerts to configured channels |

**Exam note:** Falco = detection only. It never blocks a syscall or kills a process natively — that requires external response tooling (falcosidekick, Falco Talon) or a separate preventive control (AppArmor, seccomp, OPA Gatekeeper).

### Rule Structure (required fields)
```yaml
- rule: Shell Spawned in Container
  desc: Detect shell spawned inside a running container
  condition: >
    spawned_process and container.id != host and proc.name in (shell_binaries)
  output: >
    Shell in container (user=%user.name container=%container.name
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline)
  priority: WARNING
  tags: [container, shell, mitre_execution]
```
Required: `rule, desc, condition, output, priority`. Optional: `tags`.

### Condition Operators
| Operator | Use |
|---|---|
| `and / or / not` | Boolean logic |
| `in (list)` | Match against a defined list |
| `startswith` | Path/string prefix match |
| `contains / icontains` | Substring match (case-sensitive/insensitive) |
| `exists` | Field is present |

### Priority Levels (highest → lowest)
Emergency > Alert > Critical > Error > **Warning** > Notice > Informational > Debug
(Default shell-in-container rule = WARNING)

### Output Channels
| Channel | Use case |
|---|---|
| stdout | Default, container logs |
| file | Local log file |
| syslog | Central syslog aggregation |
| program | Pipe to external script |
| http/https | Webhook to external service |
| **gRPC** | Structured stream — falcosidekick, custom clients |

### Custom Rules Best Practice
- Never edit shipped `falco_rules.yaml` directly.
- Add custom rules in a separate file (rules directory), loaded/appended via `falco.yaml`.
- Override an existing default rule by re-declaring same `rule:` name in a later-loaded file to change fields like `priority`.
- Validate before deploy: `falco --validate <file.yaml>`

### Detection vs Prevention (exam gotcha)
| Tool | Type | Action |
|---|---|---|
| Falco | Detection | Alerts on anomalous runtime behavior |
| AppArmor/seccomp | Prevention | Blocks disallowed syscalls before execution |
| OPA Gatekeeper | Prevention | Blocks non-compliant objects at admission time |

**Common trap:** Exam distractors imply Falco 'blocks' malicious activity — it does not, by design.
