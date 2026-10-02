# Chapter 9 Lab: Supply Chain Security II: Static Analysis, SBOM & Signing

**Estimated time:** 35 minutes

## Exercise

Run kubesec scan on a deployment manifest to identify risky settings, sign a container image using cosign with a generated keypair, then write a Kyverno policy requiring verifyImages before the image is allowed to run

## Key takeaway

Supply chain artifact validation tasks require you to distinguish manifest-level static analysis (kubesec) from image signing/verification (cosign/Sigstore), and enforce signing at admission time.
