# Chapter 5 Lab: Microservice Security I: Pod Security Admission & Policy Engines

**Estimated time:** 35 minutes

## Exercise

Label a namespace with pod-security.kubernetes.io/enforce: restricted, attempt to deploy a privileged pod and observe the rejection, then write a Kyverno ClusterPolicy that blocks containers running as root

## Key takeaway

Microservice vulnerability tasks require correctly labeling namespaces for PSA and writing enforce-mode policies in Kyverno or OPA Gatekeeper, not just auditing them.
