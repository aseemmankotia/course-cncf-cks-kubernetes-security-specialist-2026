# System Hardening Cheat Sheet (CKS)

## Seccomp
| Type | Behavior |
|---|---|
| Unconfined | No syscall filtering (avoid) |
| RuntimeDefault | Uses container runtime's built-in default profile |
| Localhost | Uses custom JSON profile file on node |

**Profile location:** `/var/lib/kubelet/seccomp/<profile>.json` (path relative to this root for `localhostProfile`).

**Pod snippet:**
```yaml
securityContext:
  seccompProfile:
    type: Localhost
    localhostProfile: my-profile.json
```

**Custom profile skeleton:**
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "syscalls": [
    {"names": ["read","write","exit"], "action": "SCMP_ACT_ALLOW"}
  ]
}
```
Profile file must exist on **every node** the pod may schedule to.

## AppArmor
| K8s Version | Method |
|---|---|
| < 1.30 (beta) | Annotation: `container.apparmor.security.beta.kubernetes.io/<container>: localhost/<profile>` |
| >= 1.30 (GA) | `securityContext.appArmorProfile.type: Localhost` + `localhostProfile: <profile>` |

**Load profile on node:**
```bash
apparmor_parser -q -r /etc/apparmor.d/k8s-nginx
aa-status   # verify loaded profiles
```
Profiles live in `/etc/apparmor.d/`. Must load on node BEFORE pod scheduling or pod fails to start (FailedCreatePodSandBox).

## Kernel Module Hardening
| Goal | Method |
|---|---|
| Block auto-load only | `blacklist <module>` in `/etc/modprobe.d/blacklist.conf` |
| Block ALL loading (incl. manual modprobe) | `install <module> /bin/true` in `/etc/modprobe.d/` |

Common exam targets: `dccp`, `sctp`, `cramfs`, `freevxfs`.

## SSH / Node Access
- `PermitRootLogin no` → blocks root login only (not full lockdown).
- Full lockdown: `systemctl disable --now sshd && systemctl mask sshd` + route via bastion/jump host.
- Prefer short-lived, MFA-audited access over standing SSH keys.

## Reducing Attack Surface
- Remove unused packages: `apt purge <pkg>` / `yum remove <pkg>`.
- Disable unneeded services: `systemctl disable --now <service>` (e.g., cups, avahi-daemon, telnet).
- Least-privilege IAM: node instance roles should NOT have broad cloud-admin permissions; scope to only what kubelet/CNI/CSI require.
- Combine seccomp + AppArmor + minimal packages = defense in depth (exam tests this layering, not one control alone).

## Exam Traps
1. Forgetting `localhost/` prefix in AppArmor beta annotation → profile silently ignored.
2. Using `blacklist` when manual `modprobe` block is required → use `install ... /bin/true` instead.
3. Custom seccomp/AppArmor profile missing on a node the pod schedules to → CreateContainerError.
