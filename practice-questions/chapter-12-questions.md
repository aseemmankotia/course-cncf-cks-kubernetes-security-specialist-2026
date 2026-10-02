## Chapter 12: Full CKS Exam Simulation & Time-Management Strategy — Practice Questions

**Q1.** You are 45 minutes into the 120-minute CKS exam and have fully completed only 6 of 16 tasks, several of which were low point-value. What is the BEST next action?

A) Continue working tasks in their original order regardless of difficulty
B) Stop and re-verify the 6 completed tasks in detail before continuing
C) Restart the exam session to reset the timer
D) Skip ahead and complete the easiest-looking remaining tasks first to lock in points, then return to harder ones with remaining time

<details>
<summary>Answer</summary>

**Correct: D**

Locking in guaranteed points from easy tasks first maximizes score before time pressure forces rushed or skipped attempts on remaining items.

**Why the others are wrong:**
- Rigid ordering ignores difficulty signals and risks running out of time on easy points sitting later in the list
- Re-verifying already-completed work spends time better used attempting unstarted tasks
- Restarting is not possible in the real exam and would forfeit the attempt

</details>

---

**Q2.** Each CKS task specifies its own context, e.g. 'kubectl config use-context this-cluster'. What should you do FIRST upon opening the task?

A) Run kubectl get nodes to inspect the default cluster
B) Run kubectl config use-context this-cluster before making any changes
C) Skip it since the default context usually matches the task
D) SSH directly into a node and ignore kubectl context

<details>
<summary>Answer</summary>

**Correct: B**

Solving a task on the wrong context/cluster yields zero credit even if the YAML/config is technically correct, so switching context is always the mandatory first step.

**Why the others are wrong:**
- Inspecting nodes on the default (wrong) context wastes time and doesn't fix the context mismatch
- Assuming defaults match is a leading cause of zero-scored tasks in CKS
- SSH is unnecessary for context switching and bypasses the required kubectl workflow

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** Which command confirms the currently active kubectl context so you can verify you're working on the correct cluster before making changes?

A) kubectl cluster-info
B) kubectl config view
C) kubectl config current-context
D) kubectl get context

<details>
<summary>Answer</summary>

**Correct: C**

kubectl config current-context prints only the active context name, giving a fast one-line confirmation before editing resources.

**Why the others are wrong:**
- kubectl cluster-info shows endpoint/service info, not which context is active
- kubectl config view dumps the full kubeconfig, requiring extra parsing to find the active context
- kubectl get context is not a valid kubectl command

</details>

---

**Q4.** You've spent 12 minutes on a low-value NetworkPolicy task and still have a YAML syntax error you can't locate. What should you do?

A) Keep debugging until the error is resolved no matter how long it takes
B) Delete all progress and skip the task entirely with nothing applied
C) Flag the task, apply whatever partial/valid config you have, and move to the next task
D) Restart your whole exam session to get a clean slate

<details>
<summary>Answer</summary>

**Correct: C**

Applying a partially correct manifest before moving on preserves any partial-credit sub-checks (e.g., correct selector even if ports are wrong), and flagging lets you return if time permits.

**Why the others are wrong:**
- Sinking more time into one low-value task risks losing points on unattempted higher-value tasks later
- Leaving nothing applied guarantees zero credit even for parts you got right
- Session restarts are not available and would forfeit the entire attempt

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** What does 'partial credit' mean for a multi-step CKS task such as configuring a NetworkPolicy with specific selectors and ports?

A) The task is all-or-nothing; any single mistake zeroes the entire task
B) Only the final verification command run by the grader matters, not the resource itself
C) Partial credit applies only to the overall exam score, never to individual tasks
D) Each sub-requirement is graded independently, so completing some correctly earns partial points even if the task isn't 100% correct

<details>
<summary>Answer</summary>

**Correct: D**

CKS grading scripts typically check discrete sub-conditions per task, so a resource that's created correctly but has one wrong field can still earn points for the parts done correctly.

**Why the others are wrong:**
- Treating tasks as all-or-nothing causes candidates to abandon tasks they could have partially scored
- The resource state itself is what's checked, not a single grader command in isolation
- Partial credit operates at the task/sub-check level, which is exactly what rolls up into the overall score

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** Which of the following is an allowed reference source during the CKS exam?

A) kubernetes.io official documentation and official docs for curriculum tools like Falco or Cilium
B) Stack Overflow discussion threads
C) A personal cheat sheet saved locally on your machine
D) An AI chat assistant opened in another tab

<details>
<summary>Answer</summary>

**Correct: A**

The exam permits only the official Kubernetes documentation and the official docs of tools explicitly covered in the CKS curriculum, accessed through the proctored browser.

**Why the others are wrong:**
- Third-party Q&A sites like Stack Overflow are not on the approved documentation list and violate exam rules
- External personal notes or local files are prohibited under proctoring rules
- AI assistants are not permitted reference tools during the proctored exam

</details>

---

**Q7.** For a 120-minute CKS exam with roughly 15-16 tasks, what average time budget per task leaves a reasonable buffer for review?

A) About 6-8 minutes per task
B) About 15 minutes per task
C) About 2 minutes per task
D) Divide time evenly with no buffer reserved

<details>
<summary>Answer</summary>

**Correct: A**

An even split of 120 minutes across ~15-16 tasks is roughly 7-8 minutes each; targeting 6-8 minutes per task leaves 10-15 minutes reserved for flagged-task revisits and final verification.

**Why the others are wrong:**
- 15 minutes per task would only allow 8 tasks to be attempted, leaving many unstarted
- 2 minutes is unrealistically short for tasks requiring YAML edits and verification
- No buffer means a single stuck task can derail completion of the remaining tasks

</details>

---

**Q8.** Immediately after applying your solution for a task, what should you always do before moving to the next task?

A) Immediately switch context to the next task without checking anything
B) Refresh the browser to clear the terminal session
C) Delete the resource you just created to free up cluster memory
D) Verify the resource state with kubectl get/describe or an equivalent check

<details>
<summary>Answer</summary>

**Correct: D**

kubectl apply can succeed on syntactically valid but logically wrong YAML (wrong selector, wrong namespace); a quick get/describe check catches these silent failures before points are lost.

**Why the others are wrong:**
- Skipping verification risks carrying forward a broken or incomplete task with zero credit
- Refreshing the browser does not confirm resource correctness and wastes time
- Deleting a just-created resource would erase the work you need graded

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

