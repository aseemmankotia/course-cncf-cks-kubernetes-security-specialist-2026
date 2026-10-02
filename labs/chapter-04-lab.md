# Chapter 4 Lab: System Hardening: AppArmor, Seccomp & Reducing Attack Surface

**Estimated time:** 30 minutes

## Exercise

Load a custom seccomp profile restricting syscalls for a pod, apply an AppArmor profile via annotation on a kind node, and confirm blocked syscalls produce the expected denial in container logs

## Key takeaway

System Hardening tests kernel-level controls (seccomp, AppArmor) and requires you to remember they are enforced at the host/node level, not just in the pod spec.
