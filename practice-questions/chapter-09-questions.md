## Chapter 9: Supply Chain Security II: Static Analysis, SBOM & Signing — Practice Questions

**Q1.** You run `kubesec scan deployment.yaml` against a manifest that sets `allowPrivilegeEscalation: true`, has no `seccompProfile`, and grants the `SYS_ADMIN` capability. What will kubesec report?

A) A negative or low score with advise/critical entries listing each risky field and remediation guidance
B) A passing score because kubesec only checks image vulnerabilities, not pod security context
C) An error, because kubesec cannot parse Deployment objects, only Pod objects
D) A score of exactly 0 with no further detail, since kubesec does not explain findings

<details>
<summary>Answer</summary>

**Correct: A**

kubesec assigns positive points for hardening settings and negative points for risky ones (privilege escalation, missing seccomp, dangerous capabilities); the JSON output includes `scoring.critical` and `scoring.advise` arrays with the field and a human-readable reason.

**Why the others are wrong:**
- kubesec's core purpose is scoring the security posture of the manifest itself, not scanning image contents
- kubesec scans any workload manifest containing a PodSpec, including Deployments, DaemonSets, and Pods
- kubesec always returns structured scoring detail (critical/advise arrays), not a bare number

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team needs to produce a machine-readable inventory of every package and library inside a container image, for later vulnerability correlation and license auditing. Which tool BEST fits this requirement?

A) kubesec, to generate a package manifest from the image layers
B) cosign, to embed a package list inside the image signature
C) kube-bench, to audit installed packages against CIS benchmarks
D) syft, to generate an SBOM in SPDX or CycloneDX format from the image

<details>
<summary>Answer</summary>

**Correct: D**

syft inspects image layers and filesystems to produce a Software Bill of Materials in standard formats (SPDX, CycloneDX), which is the artifact used for downstream vuln scanning (e.g. with grype) and license auditing.

**Why the others are wrong:**
- kubesec scores Kubernetes manifest YAML for security misconfigurations, not image package contents
- cosign signs and verifies artifacts/images; it does not enumerate installed packages
- kube-bench checks node and cluster configuration against CIS Kubernetes Benchmarks, not package inventories

</details>

---

**Q3.** You have `cosign.pub` from your organization's signing key and need to verify that `registry.example.com/app:v2` was signed by that key before pulling it into production. Which command performs this check?

A) cosign sign --key cosign.pub registry.example.com/app:v2
B) cosign generate-key-pair --key cosign.pub registry.example.com/app:v2
C) cosign verify --key cosign.pub registry.example.com/app:v2
D) cosign attest --key cosign.pub registry.example.com/app:v2

<details>
<summary>Answer</summary>

**Correct: C**

`cosign verify --key <public-key> <image>` checks the image's signature against the supplied public key and, if valid, prints the verified signature payload; exit code 0 indicates success.

**Why the others are wrong:**
- `cosign sign` requires a private key to produce a signature, not a public key, and is the opposite operation
- `generate-key-pair` creates a new key pair; it takes no image argument and is unrelated to verification
- `cosign attest` attaches an in-toto attestation (e.g. SBOM or provenance) to an image; it does not perform signature verification

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** The security team wants the cluster to reject any Pod whose container image is not signed by a trusted cosign key, enforced automatically at admission time. Which configuration BEST achieves this?

A) A NetworkPolicy restricting egress to the registry so only signed images can be pulled
B) A PodSecurity admission `restricted` profile applied to the namespace
C) An OPA Gatekeeper ConstraintTemplate that checks the image tag equals `latest`
D) A Kyverno ClusterPolicy using the `verifyImages` rule with the trusted public key listed under `attestors`

<details>
<summary>Answer</summary>

**Correct: D**

Kyverno's `verifyImages` rule integrates with cosign/Sigstore: it fetches the image signature from the registry and validates it against the configured public key(s) or keyless attestors, failing admission if verification fails.

**Why the others are wrong:**
- NetworkPolicy controls network traffic, not registry pull authorization or signature validation
- PodSecurity `restricted` enforces pod security context hardening (privilege escalation, capabilities, etc.), not image signature verification
- Checking for the `latest` tag is a Dockerfile/tagging best practice, unrelated to cryptographic signature verification

</details>

---

**Q5.** In Sigstore's keyless signing workflow used by `cosign sign` (no `--key` flag), what role does Fulcio play?

A) Fulcio issues a short-lived X.509 certificate binding the signer's OIDC identity to an ephemeral signing key
B) Fulcio stores the signed SBOM alongside the image manifest in the registry
C) Fulcio is the transparency log that records every signature and certificate issued
D) Fulcio scans the image for known CVEs before allowing the signature to be created

<details>
<summary>Answer</summary>

**Correct: A**

Fulcio is the Sigstore certificate authority: given a valid OIDC token, it issues a short-lived code-signing certificate tying an ephemeral key pair to the signer's verified identity (e.g. email/CI identity), enabling keyless signing.

**Why the others are wrong:**
- SBOM storage is unrelated to Fulcio; SBOMs/attestations are attached via `cosign attest` and stored in the registry as OCI artifacts
- Rekor, not Fulcio, is the immutable transparency log recording signing events
- Vulnerability scanning is performed by tools like grype/trivy, not Fulcio

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** During static analysis of a Dockerfile, which instruction pattern is the CLEAREST supply-chain security red flag the exam expects you to identify?

A) FROM ubuntu:22.04
B) FROM myapp:latest combined with running the container process as root with no USER instruction
C) COPY app.py /app/app.py
D) EXPOSE 8080

<details>
<summary>Answer</summary>

**Correct: B**

Floating `:latest` tags break reproducibility and traceability (a core supply-chain requirement), and omitting `USER` leaves the container running as root, violating least privilege — both are classic exam gotchas for Dockerfile static analysis.

**Why the others are wrong:**
- A pinned base image tag like `ubuntu:22.04` is a good practice, not a red flag
- A simple COPY of application code is normal and not inherently insecure
- EXPOSE only documents a port for humans/tools; it has no runtime security effect by itself

</details>

---

**Q7.** You need to generate an SBOM in CycloneDX JSON format for an image and immediately scan it for known vulnerabilities without re-pulling the image. Which command sequence is correct?

A) cosign attest <image> -o cyclonedx-json > sbom.json  followed by  cosign verify sbom:./sbom.json
B) kubesec scan <image> -o cyclonedx-json > sbom.json  followed by  kubesec scan sbom:./sbom.json
C) syft <image> -o cyclonedx-json > sbom.json  followed by  grype sbom:./sbom.json
D) grype <image> -o cyclonedx-json > sbom.json  followed by  syft sbom:./sbom.json

<details>
<summary>Answer</summary>

**Correct: C**

syft generates the SBOM in the requested format from the image, and grype accepts an `sbom:` source so it can scan the already-generated file instead of re-pulling the image — the standard SBOM-then-scan pipeline.

**Why the others are wrong:**
- cosign attests/signs artifacts; it does not generate SBOMs, and `cosign verify` does not take an `sbom:` source
- kubesec only scans Kubernetes manifest YAML, not container images, and has no SBOM output mode
- The tool order is reversed: grype is the scanner, syft is the generator, so this sequence is backwards

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which statement BEST describes 'chain of custody' in a Kubernetes supply chain security context?

A) A verifiable, unbroken record of an artifact's origin and every transformation from source build through registry push to cluster deployment, typically backed by signatures and attestations
B) Encrypting etcd data at rest so secrets cannot be read by unauthorized users
C) Rotating the CA certificates used by the kube-apiserver on a fixed schedule
D) Restricting which namespaces a ServiceAccount can access via RBAC

<details>
<summary>Answer</summary>

**Correct: A**

Chain of custody means every step — source commit, build, SBOM generation, signing, registry storage, admission verification — is traceable and cryptographically attestable, so a deployed artifact can be proven to match its trusted origin.

**Why the others are wrong:**
- etcd encryption at rest protects stored cluster data, unrelated to artifact provenance
- CA certificate rotation is a cluster PKI maintenance task, not an artifact provenance concept
- RBAC namespace restriction controls authorization, not the traceability of how an artifact was built and delivered

</details>

---

