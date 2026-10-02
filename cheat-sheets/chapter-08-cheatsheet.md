## Chapter 8 Cheat Sheet — Image Footprint & Vulnerability Scanning

### Trivy Core Commands
| Command | Purpose |
|---|---|
| `trivy image <img>` | Scan a container image for CVEs (all severities by default) |
| `trivy image --severity CRITICAL,HIGH <img>` | Filter report to relevant severities |
| `trivy image --exit-code 1 --severity CRITICAL <img>` | Fail CI pipeline on CRITICAL findings |
| `trivy fs ./` | Scan filesystem / source repo (deps, IaC misconfig) |
| `trivy k8s cluster` | Scan live cluster workloads (via trivy-operator) |

### CVSS Severity Ranges
| Severity | Score Range | Remediation Priority |
|---|---|---|
| CRITICAL | 9.0–10.0 | 1st |
| HIGH | 7.0–8.9 | 2nd |
| MEDIUM | 4.0–6.9 | 3rd |
| LOW | 0.1–3.9 | 4th |

### Base Image Footprint Comparison
| Image | Shell/Pkg Mgr? | Relative Size | Attack Surface |
|---|---|---|---|
| ubuntu:latest | Yes | Large | High |
| alpine:latest | Yes (apk, sh) | Small (~5MB) | Medium |
| gcr.io/distroless/static | No | Smallest for static binaries | Lowest |
| gcr.io/distroless/base | No (has glibc) | Small | Low |

### Multi-Stage Build Pattern
```dockerfile
FROM golang:1.22 AS build
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o server .

FROM gcr.io/distroless/static-debian12
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
```
Exam trap: forgetting to change the final `FROM` — copying into the builder image again defeats the exercise.

### Admission-Control Enforcement Options
| Mechanism | Enforcement Point | Native Signature Verify? |
|---|---|---|
| ImagePolicyWebhook | kube-apiserver plugin → external backend | No |
| Gatekeeper (OPA) | ValidatingWebhook + Rego ConstraintTemplate | No (needs custom Rego) |
| Kyverno | ValidatingWebhook, native YAML rules | Yes (`verifyImages`) |

**ImagePolicyWebhook flags:** `--enable-admission-plugins=ImagePolicyWebhook` + `--admission-control-config-file=<path>` (defines `kubeConfigFile`, `defaultAllow`, `allowTTL`, `denyTTL`).

**Kyverno registry allowlist snippet:**
```yaml
spec:
  validationFailureAction: Enforce
  rules:
  - name: restrict-registry
    match: {resources: {kinds: [Pod]}}
    validate:
      pattern:
        spec:
          containers:
          - image: "registry.internal.corp/*"
```

### Exam Gotchas
- `--exit-code 0` (Trivy default) never fails a pipeline — must set `1`.
- `validationFailureAction: Audit` logs but does NOT block; only `Enforce` blocks.
- Distroless ≠ scanned — it still needs Trivy scanning; it just lacks shell/tools.
- ImagePolicyWebhook needs BOTH the enable-admission-plugins flag AND the config file — one alone does nothing.
