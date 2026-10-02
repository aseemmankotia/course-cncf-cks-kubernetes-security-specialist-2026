## Chapter 4: System Hardening: AppArmor, Seccomp & Reducing Attack Surface — Practice Questions

**Q1.** A Pod must run with the container runtime's default seccomp profile applied. Which securityContext configuration achieves this?

A) seccompProfile: {type: Unconfined}
B) seccompProfile: {type: Localhost, localhostProfile: default.json}
C) seccompProfile: {type: RuntimeDefault}
D) annotations: {seccomp.security.alpha.kubernetes.io/pod: docker/default}

<details>
<summary>Answer</summary>

**Correct: C**

type: RuntimeDefault tells the kubelet to apply the container runtime's built-in default seccomp profile (e.g., containerd's default), which blocks dangerous syscalls without needing a custom file.

**Why the others are wrong:**
- Unconfined disables seccomp filtering entirely — the opposite of hardening.
- Localhost requires a custom JSON profile to exist on the node at /var/lib/kubelet/seccomp/; the exam scenario didn't ask for a custom profile.
- This is the deprecated alpha annotation syntax removed in modern Kubernetes; securityContext.seccompProfile is the current API.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** You created a custom AppArmor profile named 'k8s-nginx' and loaded it on every node with apparmor_parser. Which pod-level configuration correctly applies it to a container named 'web' on Kubernetes 1.29 (pre-GA field)?

A) securityContext.appArmorProfile.type: Localhost
B) annotation container.apparmor.security.beta.kubernetes.io/web: k8s-nginx
C) annotation container.apparmor.security.beta.kubernetes.io/web: localhost/k8s-nginx
D) securityContext.seccompProfile.localhostProfile: k8s-nginx

<details>
<summary>Answer</summary>

**Correct: C**

Prior to AppArmor GA (1.30), profiles are applied via the pod annotation container.apparmor.security.beta.kubernetes.io/<container-name> with value localhost/<profile-name>, matching the name loaded by apparmor_parser on the node.

**Why the others are wrong:**
- appArmorProfile field only exists from Kubernetes 1.30 onward (GA); this scenario specifies 1.29.
- Missing the required 'localhost/' prefix — without it the kubelet rejects or ignores the profile reference.
- This is a seccomp field, not AppArmor — different subsystem entirely.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A cluster hardening requirement states that the 'dccp' kernel module must never be loadable on worker nodes, even manually via modprobe. What is the correct approach?

A) Add 'blacklist dccp' to /etc/modprobe.d/blacklist.conf only
B) Run 'rmmod dccp' once during node provisioning
C) Set kernel.modules_disabled=1 sysctl at runtime
D) Add 'install dccp /bin/true' to a file in /etc/modprobe.d/ to override module installation

<details>
<summary>Answer</summary>

**Correct: D**

'install dccp /bin/true' overrides the module install command so any load attempt (automatic or manual modprobe) runs a no-op instead of loading the module — this actually prevents loading, not just aliasing.

**Why the others are wrong:**
- 'blacklist' only prevents automatic loading via aliases/udev; a direct 'modprobe dccp' command still works, failing the requirement.
- rmmod only removes an already-loaded module once; it can be reloaded later unless install is overridden.
- kernel.modules_disabled=1 disables ALL module loading permanently and is irreversible without reboot — too broad and not the targeted per-module technique tested.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** Where must a custom seccomp JSON profile file be placed on a node so it can be referenced by localhostProfile: my-profile.json?

A) /etc/kubernetes/seccomp/my-profile.json
B) /var/lib/kubelet/seccomp/my-profile.json
C) /etc/apparmor.d/my-profile.json
D) /var/run/containerd/seccomp/my-profile.json

<details>
<summary>Answer</summary>

**Correct: B**

The kubelet's default seccomp root directory is /var/lib/kubelet/seccomp/; localhostProfile paths are relative to this directory.

**Why the others are wrong:**
- This is not a recognized kubelet seccomp path.
- That is the AppArmor profile directory, not seccomp.
- This is not the kubelet's seccomp root; the kubelet, not containerd directly, resolves the profile path.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A security policy requires that no administrator can directly SSH into worker nodes; all node-level access must go through an auditable jump path with MFA. Which single change BEST enforces this at the OS level?

A) Disable and mask the sshd service on worker nodes, requiring access via a bastion host with session recording
B) Set PermitRootLogin no in sshd_config on each node
C) Add worker nodes to a Kubernetes NetworkPolicy denying ingress on port 22
D) Rotate node SSH host keys weekly

<details>
<summary>Answer</summary>

**Correct: A**

Disabling/masking sshd entirely (systemctl disable --now sshd; systemctl mask sshd) removes direct SSH attack surface, forcing all access through a controlled, audited bastion — matching the requirement to eliminate direct SSH.

**Why the others are wrong:**
- PermitRootLogin no only blocks root login; non-root SSH access remains fully possible, not eliminating direct access.
- NetworkPolicy operates at the pod network level via CNI and does not affect the node's host-level sshd listening on the node's own interface.
- Key rotation improves crypto hygiene but doesn't stop direct SSH access at all.

</details>

---

**Q6.** Which command loads an AppArmor profile file into the kernel on a worker node before it can be referenced by a pod?

A) aa-status --load /etc/apparmor.d/k8s-nginx
B) kubectl apply -f k8s-nginx-apparmor.yaml
C) apparmor_parser -q -r /etc/apparmor.d/k8s-nginx
D) systemctl restart apparmor --profile k8s-nginx

<details>
<summary>Answer</summary>

**Correct: C**

apparmor_parser -q -r (quiet, replace) is the standard command to load/reload an AppArmor profile file into the kernel LSM on the node.

**Why the others are wrong:**
- aa-status only reports currently loaded profiles; it has no --load flag.
- Kubernetes has no native AppArmor profile object — profiles must be loaded on the node's OS, not applied via the API server.
- Restarting the apparmor service doesn't load a specific new profile file by itself; apparmor_parser is required.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** What is the exam-relevant difference between seccompProfile.type: RuntimeDefault and type: Localhost?

A) RuntimeDefault blocks all syscalls; Localhost allows all syscalls
B) RuntimeDefault only works with Docker; Localhost only works with containerd
C) RuntimeDefault uses the container runtime's built-in default filter; Localhost uses a custom JSON profile file stored on the node
D) They are functionally identical, just different syntax

<details>
<summary>Answer</summary>

**Correct: C**

RuntimeDefault delegates to the container runtime's bundled default seccomp filter (a broad, general-purpose deny list). Localhost lets you specify a custom, workload-tailored JSON profile file referenced by localhostProfile, providing tighter least-privilege control.

**Why the others are wrong:**
- RuntimeDefault doesn't block everything — it's a curated default profile permitting most common syscalls while blocking dangerous ones.
- Both work across runtimes supporting the OCI seccomp spec (containerd, CRI-O); this isn't a runtime-exclusivity distinction.
- They differ significantly — one is a static default, the other fully customizable per profile file.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A CKS scenario asks you to reduce the attack surface of a freshly provisioned worker node. Which action provides the MOST direct reduction in attack surface at the OS level?

A) Enable a NetworkPolicy default-deny in every namespace
B) Uninstall unused packages and disable unnecessary systemd services (e.g., cups, avahi) on the node
C) Enable audit logging on the API server
D) Apply a PodSecurity 'restricted' label to all namespaces

<details>
<summary>Answer</summary>

**Correct: B**

Removing unused packages and disabling unneeded services directly shrinks the OS-level attack surface by eliminating unnecessary running processes, listening ports, and exploitable binaries on the node itself.

**Why the others are wrong:**
- NetworkPolicy default-deny hardens pod-to-pod network traffic, not the underlying node OS.
- Audit logging improves detection/visibility but does not reduce the actual attack surface.
- PodSecurity restricted enforces pod-level security context requirements, not node OS package/service hygiene.

</details>

---

