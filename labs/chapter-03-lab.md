# Chapter 3 Lab: Cluster Hardening II: Securing the API Server & Kubelet

**Estimated time:** 30 minutes

## Exercise

Edit the kubelet config on a kind/kubeadm node to disable anonymous auth and the read-only port, restart kubelet, then confirm unauthenticated requests to the kubelet API are rejected with curl

## Key takeaway

Cluster Hardening also tests kubelet and API server configuration flags directly, so know which flags disable insecure defaults from memory.
