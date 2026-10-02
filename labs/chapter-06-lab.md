# Chapter 6 Lab: Microservice Security II: mTLS & Secrets Management

**Estimated time:** 30 minutes

## Exercise

Configure an EncryptionConfiguration resource to encrypt Secrets at rest using aescbc, restart the API server, then verify a Secret is stored encrypted in etcd using etcdctl get with hex dump

## Key takeaway

Secrets management tasks test whether you know base64 is not encryption and can correctly configure etcd-level encryption at rest and volume-based secret mounting.
