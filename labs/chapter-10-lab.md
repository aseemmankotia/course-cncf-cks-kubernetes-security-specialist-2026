# Chapter 10 Lab: Monitoring I: Falco Runtime Detection

**Estimated time:** 30 minutes

## Exercise

Install Falco on a kind/kubeadm node, trigger the default 'Terminal shell in container' rule by exec-ing into a pod, then author a custom rule that alerts when a process writes to /etc/ inside a container

## Key takeaway

Falco tasks test your ability to recognize and author syscall-level detection rules, and to know that Falco alerts rather than blocks by default.
