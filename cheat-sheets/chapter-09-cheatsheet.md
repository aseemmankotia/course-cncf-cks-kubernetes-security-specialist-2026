## Chapter 9 Cheat Sheet: Static Analysis, SBOM & Signing

### kubesec — manifest risk scoring
| Command | Purpose |
|---|---|
| `kubesec scan pod.yaml` | Score a manifest; JSON output with `scoring.critical` / `scoring.advise` |
| Positive score | Hardened (e.g. `runAsNonRoot`, dropped capabilities) |
| Negative score | Risky (e.g. `allowPrivilegeEscalation`, `SYS_ADMIN`, no seccomp) |

### Dockerfile static analysis red flags
| Pattern | Risk |
|---|---|
| `FROM image:latest` | Not reproducible/traceable |
| No `USER` instruction | Runs as root |
| `ADD` from remote URL | Untrusted, unverified content |
| Secrets in `ENV`/`ARG` | Leaked in image history |

### SBOM (syft / grype)
| Command | Purpose |
|---|---|
| `syft <image> -o spdx-json > sbom.json` | Generate SBOM (SPDX) |
| `syft <image> -o cyclonedx-json > sbom.json` | Generate SBOM (CycloneDX) |
| `grype sbom:./sbom.json` | Scan SBOM for CVEs (no re-pull) |
| `grype <image>` | Generate + scan in one step |

### cosign / Sigstore
| Command | Purpose |
|---|---|
| `cosign generate-key-pair` | Creates `cosign.key` / `cosign.pub` |
| `cosign sign --key cosign.key <image>` | Key-based signing |
| `cosign sign <image>` | Keyless signing (OIDC → Fulcio cert → Rekor log) |
| `cosign verify --key cosign.pub <image>` | Key-based verification |
| `cosign verify <image>` (keyless) | Verifies via Fulcio cert + Rekor entry |
| `cosign attest --key cosign.key --predicate sbom.json --type cyclonedx <image>` | Attach signed SBOM/provenance attestation |
| `cosign verify-attestation --key cosign.pub <image>` | Verify attestation |

### Sigstore components
| Component | Role |
|---|---|
| Fulcio | Issues short-lived cert binding OIDC identity → ephemeral key (keyless signing) |
| Rekor | Immutable transparency log of signing events |
| cosign | CLI to sign, verify, and attest OCI artifacts |

### Kyverno admission-time verification
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  rules:
  - name: check-signature
    match:
      resources:
        kinds: [Pod]
    verifyImages:
    - imageReferences: ["registry.example.com/*"]
      attestors:
      - entries:
        - keys:
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              ...
              -----END PUBLIC KEY-----
```

### Exam traps
- kubesec scans **manifests**, not image contents — don't confuse with syft/grype.
- `cosign sign` needs a **private** key; `cosign verify` needs the **public** key — reversed = wrong answer.
- Keyless signing still produces auditability via **Rekor**, not by embedding a static key.
- SBOM generation (syft) and vulnerability scanning (grype) are separate steps/tools, often tested as a pair.
- Chain of custody = traceable + attestable from source → build → registry → admission, not RBAC or encryption at rest.
