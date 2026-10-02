# Chapter 11 Lab: Monitoring II: Audit Logging & Immutable Containers

**Estimated time:** 30 minutes

## Exercise

Write an audit policy that logs RequestResponse level detail for secrets access and Metadata level for everything else, apply it to the API server, then trigger a secret read and confirm the full request/response is captured in the audit log

## Key takeaway

Runtime security tasks require precise audit policy rule ordering and correctly implementing immutable containers without breaking legitimate application writes.
