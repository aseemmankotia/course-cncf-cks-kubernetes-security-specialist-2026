# Chapter 7 Lab: Microservice Security III: Runtime Sandboxes (gVisor & Kata)

**Estimated time:** 25 minutes

## Exercise

Create a RuntimeClass referencing the gVisor (runsc) handler, deploy a pod with runtimeClassName set, and confirm via dmesg/kernel version checks inside the pod that syscalls are intercepted by the sandbox

## Key takeaway

Runtime sandbox tasks test whether you can correctly wire a RuntimeClass to an installed handler and choose gVisor vs Kata based on the isolation requirement in the scenario.
