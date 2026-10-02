## Chapter 3: Cluster Hardening II: Securing the API Server & Kubelet — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Kubelet read-only port number? | 10255. Serves an unauthenticated, unauthorized read-only API. Exam tip: disable with --read-only-port=0. |
| 2 | Kubelet secure HTTPS API port? | 10250. Requires authentication (cert or webhook) and authorization. Exam tip: this is the port kube-apiserver uses to reach kubelet. |
| 3 | Flag to fully disable the kubelet's unauthenticated read-only port | --read-only-port=0. Exam tip: setting to 0 removes the listener entirely, not just restricts it. |
| 4 | Kubelet flags to require authenticated + authorized API requests | --anonymous-auth=false --authorization-mode=Webhook. Exam tip: Webhook mode delegates the decision to kube-apiserver via SubjectAccessReview. |
| 5 | Recommended kube-apiserver --authorization-mode value | Node,RBAC. Exam tip: pair with the NodeRestriction admission plugin for full kubelet lockdown. |
| 6 | What does the NodeRestriction admission controller restrict? | A kubelet may only modify its own Node object and Pods bound to it; blocks node-restriction.kubernetes.io/ labels. Exam tip: enable via --enable-admission-plugins=NodeRestriction. |
| 7 | etcd flag requiring client certificate authentication | --client-cert-auth=true (paired with --trusted-ca-file). Exam tip: without it, any client without a cert can read/write etcd data, including Secrets. |
| 8 | etcd flag requiring peer mutual TLS | --peer-client-cert-auth=true (with --peer-cert-file/--peer-key-file/--peer-trusted-ca-file). Exam tip: separate setting from client auth — secures inter-member traffic. |
| 9 | Default etcd client port vs peer port | Client: 2379. Peer: 2380. Exam tip: know which port to firewall for client vs member communication. |
| 10 | Kubernetes version skew: how many minor versions older can kubelet be vs kube-apiserver? | Up to 3 minor versions older (n-3, current policy as of v1.28+). Exam tip: kubelet must never be newer than the API server. |
| 11 | Why must the read-only kubelet port be disabled even with RBAC configured? | Port 10255 bypasses --authorization-mode and --anonymous-auth entirely. Exam tip: RBAC settings do NOT protect this separate endpoint. |
| 12 | Authorization mode flagged on the exam as legacy/insecure to avoid | ABAC — static policy file, no dynamic updates, poor auditability. Exam tip: correct hardened answer is always RBAC (+Node for kubelets). |
