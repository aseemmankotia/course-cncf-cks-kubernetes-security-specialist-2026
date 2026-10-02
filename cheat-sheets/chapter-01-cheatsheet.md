# Chapter 1 Cheat Sheet: Cluster Setup

## NetworkPolicy Default-Deny Template
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
**Gotcha:** This blocks DNS. Add an egress rule for UDP/TCP 53 to kube-system, or pods can't resolve names.

## Allow DNS Egress Rule
```yaml
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

## Default Behavior Table
| Scenario | Ingress | Egress |
|---|---|---|
| No NetworkPolicy at all | Allow all | Allow all |
| podSelector:{} + policyTypes both, no rules | Deny all | Deny all |
| Policy exists but doesn't select a pod | Unaffected (allow all) | Unaffected (allow all) |

## kube-bench Commands
| Task | Command |
|---|---|
| Scan master only | `kube-bench run --targets=master` |
| Scan etcd only | `kube-bench run --targets=etcd` |
| Scan all | `kube-bench run --targets=master,node,etcd,policies` |
| Output statuses | PASS / FAIL / WARN / INFO |

## Key CIS Hardening Values
| Component | Setting | Required Value |
|---|---|---|
| API server | --anonymous-auth | false |
| Kubelet | --anonymous-auth | false |
| Kubelet | --read-only-port | 0 |
| etcd | --client-cert-auth | true |
| etcd | --peer-client-cert-auth | true |
| etcd data dir | permissions | 700, owner etcd:etcd |

**Static pod remediation:** edit the manifest in `/etc/kubernetes/manifests/` — kubelet auto-restarts the pod. Never `kubectl edit deployment` for control plane components.

## TLS Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  tls:
  - hosts: ["app.example.com"]
    secretName: web-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```
Secret must be type `kubernetes.io/tls` with keys `tls.crt` / `tls.key`.

## Node Metadata Protection
- Cloud metadata IP: **169.254.169.254**
- Block via NetworkPolicy egress deny rule targeting that /32.
- Prevents SSRF → cloud credential theft.

## Binary Integrity Verification
```bash
sha256sum kubeadm
# compare output to published kubeadm.sha256
```
Never install if checksums mismatch.
