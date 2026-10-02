## Chapter 7 Cheatsheet: RuntimeClass, gVisor, Kata

### RuntimeClass Object (node.k8s.io/v1)
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: gvisor      # must match containerd runtimes.<handler>
#overhead:
#  podFixed:
#    memory: "120Mi"
#    cpu: "250m"
```

### Assigning to a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sandboxed-pod
spec:
  runtimeClassName: gvisor   # references RuntimeClass metadata.name
  containers:
  - name: app
    image: myapp:1.0
```

### containerd Handler Mapping (config.toml)
```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.gvisor]
  runtime_type = "io.containerd.runsc.v1"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata]
  runtime_type = "io.containerd.kata.v2"
```

### gVisor vs Kata vs runc

| Aspect | runc | gVisor (runsc) | Kata Containers |
|---|---|---|---|
| Kernel shared? | Yes (host) | Effectively yes, but syscalls intercepted | No — own guest kernel per pod |
| Isolation mechanism | Namespaces/cgroups | User-space app kernel (Sentry) intercepts syscalls | Hardware VM (QEMU/KVM) |
| Overhead | Lowest | Moderate (per-syscall interception cost) | Highest (VM boot + memory footprint) |
| Syscall compatibility | Full | Partial — some syscalls unsupported/emulated | Full (real kernel) |
| Best for | Trusted, performance-critical workloads | Untrusted workloads needing moderate isolation with less overhead than a full VM | Untrusted workloads needing the strongest isolation, or kernel-module-dependent code |

### Key Fields
- `RuntimeClass.handler` → maps to CRI runtime name (config-side mapping).
- `Pod.spec.runtimeClassName` → references `RuntimeClass.metadata.name` (pod-side reference).
- `RuntimeClass.spec.overhead` → tells scheduler about extra resource cost of the runtime (critical for Kata VMs).

### Exam Traps
1. Confusing `handler` (on RuntimeClass) with `runtimeClassName` (on Pod) — they are different fields on different objects.
2. Forgetting `spec.overhead` for Kata, causing scheduler under-accounting and node overcommit/evictions.
3. Assuming gVisor provides full kernel-module support like Kata — it does not; Sentry is not a real kernel.
4. Applying sandboxed runtimes to trusted, syscall-heavy workloads, causing unnecessary performance loss.
5. Misspelled or unregistered `handler` value → pod creation fails (RuntimeClass/handler not found).

### Decision Rule
- Trusted + performance-critical → **runc**
- Untrusted, moderate isolation, lower overhead than full VM → **gVisor**
- Untrusted, need strongest isolation or real kernel semantics → **Kata Containers**
