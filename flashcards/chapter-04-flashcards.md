## Chapter 4: System Hardening: AppArmor, Seccomp & Reducing Attack Surface — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | seccompProfile.type values in Kubernetes securityContext? | RuntimeDefault, Unconfined, Localhost. Exam tip: RuntimeDefault = runtime's built-in filter; Localhost = custom JSON file on node; Unconfined = no filtering (avoid in hardened clusters). |
| 2 | Default kubelet directory for custom seccomp profiles? | /var/lib/kubelet/seccomp/ — localhostProfile paths are relative to this root. Exam tip: profile file must exist on EVERY node the pod could schedule to. |
| 3 | AppArmor annotation syntax pre-GA (before K8s 1.30)? | container.apparmor.security.beta.kubernetes.io/<container-name>: localhost/<profile-name>. Exam tip: must include 'localhost/' prefix or it's ignored. |
| 4 | AppArmor GA field in securityContext (Kubernetes 1.30+)? | securityContext.appArmorProfile.type: RuntimeDefault \| Localhost \| Unconfined (with localhostProfile: <name> for Localhost). Replaces the beta annotation. |
| 5 | Command to load/reload an AppArmor profile on a node? | apparmor_parser -q -r /etc/apparmor.d/<profile-file>. Exam tip: profile must be loaded on the node BEFORE the pod references it, or the pod fails to start. |
| 6 | Command to check currently loaded AppArmor profiles on a node? | aa-status (lists enforce/complain mode profiles). Exam tip: use this to verify a profile loaded before troubleshooting pod scheduling failures. |
| 7 | File used to blacklist a kernel module (soft block only)? | /etc/modprobe.d/blacklist.conf with line 'blacklist <module>'. Exam tip: blocks auto-load via udev/aliases but NOT manual 'modprobe <module>'. |
| 8 | How to fully prevent a kernel module from ever loading (even manual modprobe)? | Add 'install <module> /bin/true' to a file in /etc/modprobe.d/. Exam tip: this overrides the install command with a no-op, unlike blacklist which only stops auto-loading. |
| 9 | sshd_config setting to disable root SSH login? | PermitRootLogin no. Exam tip: does NOT stop non-root SSH access — for full lockdown, disable/mask the sshd service and route through a bastion host instead. |
| 10 | Command to fully disable and prevent sshd from starting on a hardened node? | systemctl disable --now sshd && systemctl mask sshd. Exam tip: masking prevents even manual/accidental service starts, unlike disable alone. |
| 11 | Two required fields inside a custom seccomp JSON profile? | defaultAction (e.g., SCMP_ACT_ERRNO) and syscalls (array with names + action, e.g., SCMP_ACT_ALLOW). Exam tip: defaultAction defines the fallback behavior for any syscall not explicitly listed. |
| 12 | Practical step to reduce node OS attack surface beyond kernel/seccomp/AppArmor controls? | Remove unused packages (apt purge / yum remove) and disable unnecessary services (systemctl disable --now <service>) like cups, avahi-daemon, telnet. Exam tip: fewer running services = fewer exploitable listening ports/binaries. |
