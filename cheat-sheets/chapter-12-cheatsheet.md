# CKS Exam Simulation & Time Management Cheat Sheet

## Exam Format
| Item | Value |
|---|---|
| Duration | 120 minutes |
| Task count | ~15-16 performance-based tasks |
| Passing score | 67% |
| Retake | 1 free retake included |
| Proctoring | Remote online proctor via secure browser |
| Allowed docs | kubernetes.io + official docs of curriculum tools (Falco, Cilium, gVisor, etc.) only |

## Time Budget Table
| Phase | Time |
|---|---|
| First pass (all tasks, easiest first) | ~90 min (6-8 min/task) |
| Flagged/high-value revisits | ~15-20 min |
| Final verification pass | ~10 min |

## Context-Switch Commands
| Command | Purpose |
|---|---|
| `kubectl config get-contexts` | List all contexts at exam start |
| `kubectl config use-context <name>` | MANDATORY first step per task |
| `kubectl config current-context` | Confirm active context before editing |
| `kubectl get all -A` | Quick sanity check after switching |

## Task Triage Checklist
1. Read full task, note point weight/complexity before typing anything
2. Switch context (`use-context`) — verify with `current-context`
3. Time-box attempt: ~6-8 min avg, cap hard tasks at ~10 min
4. If stuck: apply best partial answer, flag task, move on
5. Verify every completed task (`kubectl get/describe`) before advancing
6. After first pass: revisit flagged tasks by point value, highest first
7. Reserve last 10 min purely for verification, not new attempts

## Partial Credit Strategy
- Multi-step tasks (e.g., NetworkPolicy: selector + ports + namespace) are graded per sub-check
- Always apply *something* valid — zero attempts always score 0%
- Prioritize getting the resource to exist and apply cleanly before perfecting every field

## Common Traps
- Solving a task on the wrong context/cluster (zero credit even if correct)
- Spending 20+ min perfecting one low-value task
- Not re-verifying after `kubectl apply` succeeds silently on wrong config
- Forgetting to flag and return to skipped tasks before time runs out
- Referencing non-approved documentation sites (violates exam rules)
