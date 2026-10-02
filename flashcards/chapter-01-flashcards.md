## Chapter 1: Cluster Setup: NetworkPolicy, CIS Benchmarks & TLS Ingress — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | NetworkPolicy: what does `podSelector: {}` combined with policyTypes [Ingress, Egress] and no rules mean? | Default-deny-all for the namespace — selects every pod, allows nothing since no ingress/egress rules are listed. Exam tip: this is the standard 'default deny' template you must memorize. |
| 2 | kube-bench: how do you scan ONLY the etcd CIS section? | `kube-bench run --targets=etcd`. Other valid targets: master, node, policies. Exam tip: know the exact --targets flag name and values. |
| 3 | CIS Benchmark: recommended etcd data directory permission? | 700, owned by etcd:etcd. Exam tip: kube-bench FAILs this check if permissions are looser (e.g. 755). |
| 4 | API server flag to disable anonymous authentication (CIS hardening)? | --anonymous-auth=false in /etc/kubernetes/manifests/kube-apiserver.yaml. Exam tip: editing this static pod manifest auto-triggers a kubelet restart of the pod — no kubectl apply needed. |
| 5 | Kubelet flags CIS requires for hardening? | --anonymous-auth=false and --read-only-port=0. Exam tip: read-only port 10255 exposes unauthenticated metrics if left enabled. |
| 6 | What Secret type must hold a TLS cert/key for Ingress? | kubernetes.io/tls, with data keys tls.crt and tls.key. Referenced via spec.tls[].secretName in the Ingress object. |
| 7 | Cloud metadata service IP commonly targeted by SSRF attacks? | 169.254.169.254. Exam tip: block pod egress to this IP via NetworkPolicy to prevent credential theft from the instance metadata endpoint. |
| 8 | Command to verify a downloaded Kubernetes binary's integrity? | `sha256sum <binary>` then compare against the official published checksum file. Exam tip: mismatch = do not install, binary may be tampered with. |
| 9 | Default pod-to-pod traffic behavior with zero NetworkPolicy objects in the cluster? | Allow-all (flat network) — Kubernetes has no implicit default-deny. You must create policies to restrict traffic. |
| 10 | kube-bench possible result statuses per check? | PASS, FAIL, WARN, INFO. Exam tip: WARN often means manual verification is required, not an automatic failure. |
| 11 | etcd flags CIS checks for peer/client authentication? | --client-cert-auth=true and --peer-client-cert-auth=true. Exam tip: both must be true; missing either is a common kube-bench FAIL. |
| 12 | NetworkPolicy egress rule needed alongside default-deny to keep DNS working? | Allow egress UDP/TCP port 53 to the kube-system namespace (label-selected). Exam tip: forgetting this breaks DNS for every pod under default-deny. |
