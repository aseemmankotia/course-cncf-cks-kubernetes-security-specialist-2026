## Chapter 3 Cheat Sheet — API Server & Kubelet Hardening

### Kubelet Ports
| Port | Purpose | Auth Required? | Exam Action |
|---|---|---|---|
| 10255 | Read-only API | No (always unauthenticated) | Disable: `--read-only-port=0` |
| 10250 | Secure HTTPS API | Yes (cert or webhook) | Keep, enforce auth flags |

### Kubelet Auth Flags
| Flag | Secure Value | Notes |
|---|---|---|
| `--anonymous-auth` | `false` | Rejects unauthenticated requests |
| `--authorization-mode` | `Webhook` | Delegates authz to kube-apiserver via SubjectAccessReview |
| `--read-only-port` | `0` | Fully disables unauthenticated read-only API |

### API Server Authorization
| Mode | Use? | Notes |
|---|---|---|
| `Node` | Yes | Restricts kubelet-originated requests |
| `RBAC` | Yes | Standard, auditable, dynamic |
| `Webhook` | Situational | External authz service |
| `ABAC` | Avoid | Legacy, static policy file, hard to audit |

Standard hardened value: `--authorization-mode=Node,RBAC`

### NodeRestriction Admission Controller
- Enable: `--enable-admission-plugins=NodeRestriction`
- Restricts kubelet to only modify its own Node/Pod objects
- Blocks modifying `node-restriction.kubernetes.io/` labels
- Must be paired with `Node` authorization mode to be effective

### etcd Security
| Setting | Flag | Purpose |
|---|---|---|
| Client auth | `--client-cert-auth=true` + `--trusted-ca-file` | Requires clients (e.g. API server) to present valid certs |
| Peer auth | `--peer-client-cert-auth=true` + `--peer-cert-file`/`--peer-key-file`/`--peer-trusted-ca-file` | Requires mTLS between etcd members |
| Client port | 2379 | Restrict via firewall/NetworkPolicy on host |
| Peer port | 2380 | Restrict to etcd member IPs only |

Without `--client-cert-auth=true`, any network client can read Secrets directly from etcd, bypassing RBAC entirely.

### Version Skew Policy
- kubelet may be up to **3 minor versions older** than kube-apiserver (n-3, current as of v1.28+)
- kubelet must **never** be newer than kube-apiserver
- Exam pattern: given apiserver vX, minimum supported kubelet is vX-3
- Regularly apply patch/minor upgrades — outdated versions are a named exam gotcha for "Cluster Hardening"

### Top Exam Traps
1. Thinking RBAC protects port 10255 — it does not; that port ignores all auth config.
2. Confusing etcd encryption-at-rest with client-cert-auth — they solve different problems.
3. Using `AlwaysAllow` authorization mode with anonymous-auth disabled — still insecure since any authenticated user gets full access.
4. Assuming old n-2 kubelet skew policy still applies — current policy is n-3.
