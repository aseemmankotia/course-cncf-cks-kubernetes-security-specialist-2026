## Chapter 8: Supply Chain Security I: Image Footprint & Vulnerability Scanning — Practice Questions

**Q1.** A company requires that all Pods can only be scheduled if their container images come from `registry.internal.corp/*`. Which solution BEST enforces this at admission time?

A) A Kyverno ClusterPolicy with a validate rule whose pattern restricts spec.containers[].image to 'registry.internal.corp/*'
B) A NetworkPolicy that blocks egress traffic to docker.io and other public registries
C) A PodSecurityPolicy restricting the runAsUser field
D) A Trivy scan added to the CI pipeline before the image is pushed

<details>
<summary>Answer</summary>

**Correct: A**

Kyverno (or Gatekeeper) validate policies run at admission time and can pattern-match the image field, rejecting any Pod whose image doesn't match the approved registry prefix — this is the actual enforcement point in-cluster.

**Why the others are wrong:**
- NetworkPolicy controls network traffic, not what image reference a Pod spec declares — an already-pulled or cached image would still run.
- PodSecurityPolicy is deprecated and controls security context fields, not registry sourcing.
- CI scanning happens before the cluster ever sees the Pod — it doesn't stop someone from applying a manifest with an untrusted image directly against the API server.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** Which final base image choice BEST minimizes the attack surface of a compiled Go microservice while still passing a CKS exam scenario on image footprint?

A) ubuntu:latest
B) alpine:latest with bash and curl installed
C) gcr.io/distroless/static
D) debian:bullseye-slim with the Go toolchain left in the final image

<details>
<summary>Answer</summary>

**Correct: C**

distroless/static contains no shell, no package manager, and no OS utilities — ideal for a statically linked Go binary, drastically reducing CVE surface and post-compromise tooling.

**Why the others are wrong:**
- ubuntu:latest carries a full OS with shell, package manager, and many CVE-prone packages.
- Installing bash/curl on alpine defeats the minimalism purpose and reintroduces attacker tooling.
- Leaving build tools (like the Go compiler) in the runtime image bloats the image and gives an attacker a compiler/toolchain if compromised — this is exactly the multi-stage build anti-pattern the exam tests.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A CI pipeline must fail the build if any CRITICAL vulnerabilities are found in an image before it is pushed to the registry. Which command accomplishes this?

A) trivy image --severity CRITICAL --exit-code 0 myapp:latest
B) trivy image --severity CRITICAL --exit-code 1 myapp:latest
C) trivy fs --severity CRITICAL ./
D) docker scan myapp:latest

<details>
<summary>Answer</summary>

**Correct: B**

--exit-code 1 makes Trivy return a non-zero exit status when matching-severity vulnerabilities are found, which fails the CI job. --severity CRITICAL scopes the check.

**Why the others are wrong:**
- --exit-code 0 is the default and never fails the pipeline, even when CRITICAL CVEs exist.
- trivy fs scans the filesystem/source repo, not the built container image artifact.
- docker scan is not the Trivy-based workflow tested on the CKS exam and is not consistently available in modern Docker CLI installs.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** You need to enforce that Pods will only run if their container images have been signed with Cosign. Which tool/feature is BEST suited?

A) Gatekeeper with a generic ConstraintTemplate written in Rego
B) Kyverno ClusterPolicy using the verifyImages rule
C) The built-in ImagePolicyWebhook admission controller
D) A NetworkPolicy restricting egress to the image registry

<details>
<summary>Answer</summary>

**Correct: B**

Kyverno has a native verifyImages rule designed specifically to validate Cosign/Notary signatures on container images at admission time — this is the purpose-built solution.

**Why the others are wrong:**
- Gatekeeper/OPA can technically be extended for this, but it requires custom Rego and external key management logic — not a built-in feature, making it a worse fit than Kyverno's native capability.
- ImagePolicyWebhook forwards image details to an external backend for an allow/deny decision but does not natively perform signature verification logic itself.
- NetworkPolicy has nothing to do with image signature verification.

</details>

---

**Q5.** According to CVSS scoring used by Trivy, which severities should be prioritized FIRST for remediation?

A) CRITICAL then HIGH
B) LOW then MEDIUM
C) Only CRITICAL, ignore everything else
D) All severities should be treated with equal urgency

<details>
<summary>Answer</summary>

**Correct: A**

CRITICAL (CVSS 9.0-10.0) and HIGH (7.0-8.9) represent the most exploitable/impactful vulnerabilities and are the standard remediation priority order tested on the exam.

**Why the others are wrong:**
- LOW/MEDIUM are lower priority and should be addressed after CRITICAL/HIGH, not first.
- Ignoring HIGH severities entirely leaves significant exploitable risk unaddressed.
- Treating all severities equally ignores CVSS-based risk prioritization, which is the core exam concept.

</details>

---

**Q6.** Which combination of kube-apiserver flags correctly enables the ImagePolicyWebhook admission controller?

A) --enable-admission-plugins=ImagePolicyWebhook and --admission-control-config-file=<path>
B) --admission-control-config-file=<path> only
C) --feature-gates=ImagePolicyWebhook=true
D) --authorization-webhook-config-file=<path>

<details>
<summary>Answer</summary>

**Correct: A**

ImagePolicyWebhook must be added to --enable-admission-plugins AND the apiserver needs --admission-control-config-file pointing to an AdmissionConfiguration with the webhook's kubeConfigFile, defaultAllow, allowTTL, and denyTTL settings.

**Why the others are wrong:**
- The config file alone does nothing if the plugin isn't enabled via --enable-admission-plugins.
- ImagePolicyWebhook is an admission plugin, not a feature gate — there is no such feature gate flag.
- --authorization-webhook-config-file configures the authorization webhook (a different subsystem), not image admission policy.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which Dockerfile snippet correctly implements a multi-stage build producing a minimal runtime image for a Go binary?

A) FROM golang:1.22 AS build
RUN go build -o server .
FROM gcr.io/distroless/static-debian12
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
B) FROM golang:1.22 AS build
RUN go build -o server .
FROM golang:1.22
COPY --from=build /app/server /server
CMD ["/server"]
C) FROM golang:1.22
RUN go build -o server .
CMD ["/server"]
D) FROM ubuntu:latest
RUN apt-get install -y golang && go build -o server .
CMD ["/server"]

<details>
<summary>Answer</summary>

**Correct: A**

This copies only the compiled binary from the builder stage into a distroless runtime image, eliminating the compiler, package manager, and shell from the final image.

**Why the others are wrong:**
- This keeps the full golang:1.22 build toolchain (compiler, apt, shell) in the final runtime image — no attack surface reduction achieved despite being 'multi-stage' in name.
- This is a single-stage build; the entire build image (compiler, OS tools) ships in production.
- This installs a full compiler toolchain inside ubuntu and never separates build from runtime — worst footprint of all options.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the primary security benefit of a distroless image compared to a minimal OS-based image like alpine?

A) Distroless images are always smaller in size than alpine images
B) Distroless images have no shell or package manager, limiting post-compromise tooling available to an attacker
C) Distroless images pull faster from any registry
D) Distroless images come with built-in vulnerability scanning

<details>
<summary>Answer</summary>

**Correct: B**

The defining security property of distroless is the absence of a shell, package manager, or OS utilities — an attacker who compromises the container has no way to execute follow-on commands or install tools.

**Why the others are wrong:**
- Size differences vary by language/runtime; alpine can sometimes be comparably small — size isn't the defining security property tested.
- Pull speed depends on registry/network, not the distroless property itself.
- Distroless images do not include scanning capability — scanning is a separate tool (Trivy) applied externally.

</details>

---

