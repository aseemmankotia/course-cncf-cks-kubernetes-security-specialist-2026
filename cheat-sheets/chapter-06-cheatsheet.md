# Ch.6 Cheat Sheet — mTLS & Secrets Management

## Domain
Minimize Microservice Vulnerabilities (20% of exam)

## EncryptionConfiguration Providers
| Provider | Key Location | Notes |
|---|---|---|
| identity | N/A | No encryption — plaintext. Must NOT be first in list if encryption required |
| aescbc | Local config file | Common exam default; key on control-plane disk |
| secretbox | Local config file | XSalsa20-Poly1305 |
| aesgcm | Local config file | Avoid — key reuse limit risk, requires manual rotation |
| kms (KMSv2) | External KMS plugin | Production-recommended; envelope encryption |

## Enabling Encryption at Rest
```bash
# 1. Create /etc/kubernetes/enc/enc.yaml with providers list
# 2. Add flag to kube-apiserver static pod manifest:
--encryption-provider-config=/etc/kubernetes/enc/enc.yaml
# 3. Mount the enc/ directory as a hostPath volume in the manifest
# 4. Re-encrypt EXISTING secrets:
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

## Secrets: Volume vs Env Var
| Aspect | Volume Mount | Env Var |
|---|---|---|
| Visible via `kubectl describe pod` | No (only name) | Yes (if not marked secret) |
| Readable via /proc/<pid>/environ | No | Yes |
| Leaks into logs/crash dumps | Low risk | High risk |
| Auto-updates on Secret change | Yes (kubelet sync) | No — needs pod restart |
| Exam recommendation | Preferred | Avoid |

## RBAC — Restrict Secret Access
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-db-creds
  namespace: prod
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["db-creds"]
  verbs: ["get"]
```
Bind via RoleBinding to a specific ServiceAccount only. `resourceNames` does NOT work with `list`/`watch`.

## External Secret Stores
- **Secrets Store CSI Driver**: mounts secrets from Vault/AWS Secrets Manager/Azure Key Vault directly as a volume; secret never persists in etcd.
- Optional `secretObjects` sync mirrors into a native K8s Secret for legacy consumers.
- Rotation: update value in external store → CSI driver re-syncs volume automatically.

## Pod-to-Pod mTLS Approaches
| Approach | Layer | Sidecar Needed? |
|---|---|---|
| Service mesh (Istio PeerAuthentication STRICT) | L7 | Yes |
| Cilium transparent encryption (WireGuard/IPsec) + identity-aware CiliumNetworkPolicy | Network/CNI | No |

## Common Exam Traps
- Thinking base64 = encryption (it's just encoding).
- Forgetting to re-encrypt pre-existing Secrets after enabling EncryptionConfiguration.
- Using ClusterRole/ClusterRoleBinding for secrets when namespace-scoped Role + resourceNames is the least-privilege answer.
- Assuming env-var secrets auto-update like volume mounts do.
