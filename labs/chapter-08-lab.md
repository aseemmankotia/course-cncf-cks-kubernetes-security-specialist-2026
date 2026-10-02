# Chapter 8 Lab: Supply Chain Security I: Image Footprint & Vulnerability Scanning

**Estimated time:** 35 minutes

## Exercise

Scan a sample image with Trivy, identify a HIGH severity CVE, rebuild using a distroless base image to eliminate it, then write a Kyverno policy that blocks any pod referencing an image outside an approved registry list

## Key takeaway

Supply chain image tasks require pairing vulnerability scanning (Trivy) with admission-time registry enforcement — one control does not substitute for the other.
