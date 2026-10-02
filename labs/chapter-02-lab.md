# Chapter 2 Lab: Cluster Hardening I: RBAC Least Privilege & Service Accounts

**Estimated time:** 30 minutes

## Exercise

Create a namespaced Role granting get/list/watch on pods only, bind it to a dedicated service account, disable token auto-mount on the deployment, and verify permissions with kubectl auth can-i --as

## Key takeaway

Cluster Hardening RBAC tasks test whether you can scope permissions to the exact verbs and resources needed, never more, and verify them programmatically.
