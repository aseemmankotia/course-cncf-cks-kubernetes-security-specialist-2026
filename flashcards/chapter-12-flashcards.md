## Chapter 12: Full CKS Exam Simulation & Time-Management Strategy — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | CKS exam duration and passing score? | 120 minutes total, 67% passing threshold. Exam tip: budget ~6-8 min/task, reserve 10-15 min buffer for flagged-task revisits and final verification. |
| 2 | Command to switch kubectl context at the start of each CKS task? | kubectl config use-context <context-name> — run this FIRST every task; solving the wrong cluster earns zero credit regardless of correctness. |
| 3 | Command to confirm the currently active kubectl context mid-task? | kubectl config current-context — quick one-line check before editing resources to confirm you're on the right cluster. |
| 4 | Command to list all available kubectl contexts? | kubectl config get-contexts — useful at exam start to preview all cluster names you'll switch between across tasks. |
| 5 | What documentation sources are allowed during the CKS exam? | Only kubernetes.io official docs and the official docs of curriculum tools (e.g., Falco, Cilium, gVisor). No Stack Overflow, personal notes, or AI assistants. |
| 6 | Recommended average time per task on a 120-min CKS exam with ~15-16 tasks? | 6-8 minutes per task, leaving a 10-15 min buffer at the end for flagged-task revisits and verification pass. |
| 7 | Strategy when you're unsure about a task's solution mid-attempt? | Flag it in the exam UI, apply your best partial solution first, then move on. Only return to flagged tasks after finishing confident ones. |
| 8 | Why verify every task before moving to the next? | kubectl apply can succeed on YAML that's syntactically valid but logically wrong (bad selector/namespace/port). Always run kubectl get/describe to confirm actual state before advancing — silent misconfigurations cost full task credit. |
| 9 | What is CKS 'partial credit' for multi-step tasks? | Tasks are often graded on discrete sub-checks (e.g., NetworkPolicy exists + correct selector + correct ports). Complete every sub-step even if unsure of the last — partial completion still scores points. |
| 10 | Best triage order for tasks on first pass through the exam? | Attempt tasks roughly in order of confidence/speed first to lock in guaranteed points, flagging slow or uncertain ones to revisit later rather than solving strictly in given order. |
| 11 | Why should you avoid perfecting one task for 15+ minutes? | An unattempted task scores 0%, while a rushed-but-mostly-correct task can still earn 60-80% via partial credit. Cap effort at ~10 min, flag, and move on. |
| 12 | What should you do with remaining time after finishing all first-pass attempts? | Return to flagged low-confidence tasks in order of point value, then use any final minutes to re-verify completed tasks with kubectl get/describe. |
