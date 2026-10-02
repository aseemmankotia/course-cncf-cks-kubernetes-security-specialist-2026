# Chapter 1 Lab: Cluster Setup: NetworkPolicy, CIS Benchmarks & TLS Ingress

**Estimated time:** 35 minutes

## Exercise

Deploy a default-deny NetworkPolicy plus a scoped allow rule in a kind cluster with Calico, then run kube-bench against the control-plane node and remediate two failed checks

## Key takeaway

Cluster Setup tasks test whether you can restrict network access at the namespace level and prove control-plane configuration meets CIS benchmark requirements using kube-bench.
