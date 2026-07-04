# 🎤 Docker Interview Questions — Scenario-Based

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

These test judgment, not recall — how you handle a real conversation with a teammate, not just whether you know a definition.

---

### 1. A teammate says "let's just use a bigger VM instead of containers, it's simpler." How do you respond?

**Answer:**
Start by agreeing with the part that's actually true: a VM *is* conceptually simpler — fewer moving parts, a well-understood operational model, no need to learn Docker or Kubernetes. That's a legitimate point, not something to dismiss.

Then reframe it as a trade-off question rather than a right/wrong one. A bigger VM doesn't solve the underlying problem on its own — if the team deploys straight onto a shared VM without a strict, repeatable process, the same "works on my machine" drift can creep back in at the VM level. It also doesn't give you cheap per-service isolation: spinning up a full VM per service costs minutes to boot, real memory/OS-licensing overhead, and doesn't scale down or up quickly.

The right move in an interview (and in real life) is to ask a follow-up question instead of arguing: *"What are we actually trying to avoid — complexity, or losing isolation between services?"* If it's a genuinely small, single-team, low-scale system with no need to scale services independently, one or two well-managed VMs might really be the pragmatic choice — Docker isn't a mandatory default. But if there's any expectation of scaling multiple services independently, deploying frequently, or running in a shared cluster, containers give a far better cost-to-isolation ratio.

> 🎯 **What this tests:** Whether you're dogmatic ("containers are always better") or you reason from trade-offs and ask the right clarifying question first. Interviewers are listening for judgment, not a rehearsed pro-container speech.

---

### 2. During a production incident, a teammate suggests: "Let's just exec into the running container, tweak the config, and restart the process — we'll fix the Dockerfile properly later." How do you respond?

**Answer:**
Push back on the shortcut, but by explaining the actual failure mode, not just "we don't do that." A container's entire value proposition is that the *image* is the single source of truth — what's running is supposed to be an exact, reproducible instance of it. The moment you `exec` in and hand-edit a running container, that guarantee is gone: the fix now lives only inside that one container's writable layer, which disappears the instant it's restarted, rescheduled onto another node, or replaced during a routine rolling deploy or autoscale event — all things that happen on their own, outside your control. You end up with a fleet where some replicas silently have the fix and others don't, no audit trail of what changed, and a registry image that's now lying about what's actually running.

The right move under pressure is still to fix it through the image: patch the source, rebuild, push, redeploy. If the real objection is speed, that's a pipeline problem to solve — a fast-tracked emergency deploy path — not a reason to bypass the image. The one legitimate exception worth naming: `exec`-ing in *read-only*, purely to inspect logs or state to diagnose the problem faster, is fine. The line is diagnosis versus mutation, not "never touch a running container."

> 🎯 **What this tests:** Whether you hold the immutable-infrastructure principle under real incident pressure, or cave to "just this once." A weak answer either agrees to the shortcut or refuses without explaining *why* the fix would vanish — a strong answer names the concrete failure mode and still offers a path to move fast safely.
