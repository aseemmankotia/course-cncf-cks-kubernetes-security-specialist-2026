## Chapter 7: Microservice Security III: Runtime Sandboxes (gVisor & Kata) — Practice Questions

**Q1.** A platform team allows external developers to upload arbitrary container images that run on shared multi-tenant nodes. Which configuration BEST reduces the risk of a container escaping to the host kernel?

A) Create a RuntimeClass with handler set to gvisor and assign it via spec.runtimeClassName on untrusted pods
B) Apply a strict seccomp profile with runc as the default runtime
C) Enforce the Pod Security Standards 'restricted' profile on the namespace
D) Apply a default-deny NetworkPolicy to the namespace

<details>
<summary>Answer</summary>

**Correct: A**

gVisor intercepts syscalls in a user-space kernel (Sentry), preventing untrusted containers from directly reaching the host kernel — the specific threat of unknown/arbitrary images.

**Why the others are wrong:**
- Seccomp restricts which syscalls runc containers can make, but the container still shares the host kernel directly — it does not add an isolation boundary against kernel exploits.
- Pod Security Standards constrain pod security context capabilities (privileged, hostPath, etc.) but do not sandbox the syscall interface to the kernel.
- NetworkPolicy controls network traffic between pods; it has no effect on kernel-level container escape risk.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A workload requires loading custom kernel modules inside the container and needs the strongest possible isolation boundary from the host kernel. Which runtime satisfies this requirement?

A) gVisor (runsc)
B) runc with an AppArmor profile
C) runc with a seccomp profile
D) Kata Containers

<details>
<summary>Answer</summary>

**Correct: D**

Kata Containers runs each pod inside a lightweight VM with its own guest kernel (via a hypervisor such as QEMU/KVM), so the workload has a real, separate kernel and the host kernel is never shared.

**Why the others are wrong:**
- gVisor implements a user-space application kernel that intercepts syscalls — it does not provide a real Linux kernel, so kernel module loading is not supported.
- AppArmor restricts file/resource access for processes on the shared host kernel; it does not provide a separate kernel or true VM isolation.
- Seccomp filters syscalls available to a runc container but still shares the host kernel directly.

</details>

---

**Q3.** Which field in a RuntimeClass object tells Kubernetes which underlying CRI-configured runtime to use?

A) spec.runtimeClassName
B) handler
C) spec.overhead
D) metadata.name

<details>
<summary>Answer</summary>

**Correct: B**

The 'handler' field must match the runtime name configured in the container runtime (e.g., a runtimes.<handler> entry in containerd's config.toml).

**Why the others are wrong:**
- spec.runtimeClassName is a field on the Pod spec, not on the RuntimeClass object itself.
- spec.overhead describes the resource overhead the runtime consumes (e.g., for VM-based runtimes) and is used by the scheduler, not for CRI mapping.
- metadata.name identifies the RuntimeClass object so pods can reference it, but it does not map to the CRI runtime — 'handler' does.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** Which pod spec field is used to assign a previously created RuntimeClass to a pod?

A) spec.runtimeClassName
B) spec.containers[].runtime
C) spec.securityContext.runtimeClass
D) metadata.annotations.runtimeClass

<details>
<summary>Answer</summary>

**Correct: A**

spec.runtimeClassName on the Pod object references the metadata.name of an existing RuntimeClass, causing the kubelet to invoke the mapped CRI handler for that pod.

**Why the others are wrong:**
- There is no such field as spec.containers[].runtime in the Pod API; runtime selection is pod-scoped, not per-container.
- securityContext controls privilege/capabilities settings, not runtime selection.
- Runtime assignment is a first-class spec field, not an annotation — using annotations would have no effect on runtime selection.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** Which statement BEST describes the gVisor architecture?

A) gVisor runs each container inside a full hardware-virtualized VM with its own kernel
B) gVisor is a kernel module loaded into the host kernel that filters syscalls
C) gVisor's Sentry process implements a user-space kernel that intercepts and services the container's syscalls, itself running inside a restricted sandbox
D) gVisor replaces containerd entirely and manages container images directly

<details>
<summary>Answer</summary>

**Correct: C**

gVisor's Sentry acts as an application kernel in user space: it intercepts syscalls from the container (via ptrace or a KVM-based platform) and implements them itself, reducing the syscalls that ever reach the real host kernel.

**Why the others are wrong:**
- That describes Kata Containers' VM-based model, not gVisor.
- gVisor does not modify or load anything into the host kernel — it works entirely in user space as an interception layer.
- gVisor is a CRI-compatible OCI runtime (runsc) invoked by containerd, not a replacement for containerd or an image manager.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A syscall-intensive, latency-sensitive internal database runs on trusted, internally-built images. Which runtime configuration is MOST appropriate?

A) Assign a RuntimeClass with the Kata handler for maximum isolation regardless of performance
B) Assign a RuntimeClass with the gVisor handler to reduce kernel attack surface
C) Run the workload with both gVisor and Kata handlers simultaneously for defense in depth
D) Use the default runc runtime with no RuntimeClass assigned

<details>
<summary>Answer</summary>

**Correct: D**

For trusted workloads where performance matters and syscall volume is high, standard runc avoids the syscall-interception (gVisor) or VM (Kata) overhead, since the trust boundary does not require sandboxing.

**Why the others are wrong:**
- Kata's VM isolation adds overhead inappropriate for a trusted, latency-sensitive, syscall-heavy workload where isolation is not the primary concern.
- gVisor's syscall interception adds per-syscall overhead that directly hurts a syscall-intensive workload, and the workload is trusted so sandboxing is unnecessary.
- A pod can only reference a single RuntimeClass at a time — you cannot assign two handlers to one workload.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A cluster runs a mix of trusted internal services and untrusted third-party plugin containers on the same nodes. Which approach correctly enforces different isolation levels per workload?

A) Set the container runtime's default handler to gVisor cluster-wide and require exceptions for trusted pods
B) Use a single RuntimeClass and rely on NetworkPolicy to separate trust levels
C) Taint all nodes and rely on tolerations alone to enforce isolation
D) Create separate RuntimeClass objects (e.g., 'runc' and 'gvisor') and set spec.runtimeClassName per pod based on trust level

<details>
<summary>Answer</summary>

**Correct: D**

RuntimeClass is designed exactly for this: define one RuntimeClass per handler (runtime), then reference the appropriate one per pod via spec.runtimeClassName so trusted and untrusted workloads get different isolation guarantees.

**Why the others are wrong:**
- Forcing gVisor as default for all workloads and carving out exceptions adds unnecessary overhead to trusted workloads and inverts the intended design.
- NetworkPolicy governs network traffic, not the underlying process/kernel isolation boundary — it does not address runtime-level trust separation.
- Taints/tolerations control scheduling placement, not which container runtime a pod uses.

</details>

---

**Q8.** You created a RuntimeClass with handler 'kata' and assigned it to pods that require heavy scheduling accuracy. Pods are being OOM-evicted more than expected even though container resource requests look correct. What is the MOST likely missing configuration?

A) The pod's securityContext is missing runAsNonRoot
B) The RuntimeClass handler field is misspelled
C) NetworkPolicy is blocking the kata-agent from reaching the API server
D) spec.overhead was not set on the RuntimeClass to account for the VM's additional resource consumption

<details>
<summary>Answer</summary>

**Correct: D**

Kata Containers run each pod in a VM, which consumes additional CPU/memory beyond the container's own requests. spec.overhead on the RuntimeClass tells the scheduler to account for this extra resource usage; omitting it causes nodes to be overcommitted and pods to be evicted.

**Why the others are wrong:**
- runAsNonRoot affects the container's UID, not scheduling or memory accounting — unrelated to eviction from resource overcommit.
- A misspelled handler would cause pod creation to fail outright (RuntimeClass not found), not intermittent OOM evictions.
- kata-agent communicates with the kata-shim/hypervisor components on the node, not directly with the API server, and NetworkPolicy is unrelated to VM resource accounting.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

