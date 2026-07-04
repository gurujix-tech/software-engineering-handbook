# 🎤 Docker Interview Questions — Scenario-Based

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

These test judgment, not recall — how you handle a real conversation with a teammate, not just whether you know a definition.

---

### 1. A teammate stopped a container earlier and now says "it disappeared." How do you find it?

**Answer:**
Start from the fact that `docker stop` never removes anything — it only stops the process. The container, its config, and its writable filesystem all still exist; they're just not visible in the default `docker ps`, which only lists *running* containers. The fix is simply `docker ps -a`, which lists every container regardless of state, where it'll show up with a status like `Exited (0) 10 minutes ago`.

From there it can be inspected (`docker inspect <name>`), restarted (`docker start <name>`), or its last output reviewed (`docker logs <name>`) exactly as if it had never stopped — nothing about it changed except that it isn't running. The only way a container is actually gone is an explicit `docker rm`.

> 🧠 **Remember:** `docker ps` only shows running containers — `docker ps -a` is what actually answers "did it disappear or just stop?"

> 🎯 **What this tests:** Whether you distinguish "stopped" from "removed" — the single most common source of "where did my container go" confusion for people new to Docker.

---

### 2. You've run the same `docker run` command five times while debugging the same issue. What's actually on the machine now, and how do you clean it up?

**Answer:**
`docker run` never reuses a container — every single invocation creates a brand-new one, with a new container ID and its own writable layer, even against the identical image and command. So running it five times leaves five separate containers on the machine: possibly five all still running (if none used a fixed `--name`, since Docker would otherwise refuse to reuse a name still in use), or a mix of running and exited ones if some crashed and others didn't.

To see the full damage: `docker ps -a` lists all of them. To clean up: `docker rm -f $(docker ps -aq --filter ancestor=<image>)` removes every container created from that image in one shot, or more surgically, stop and remove them by name/ID one at a time if some need to be kept. Going forward, the fix is prevention: always run debug containers with `--name` and `--rm` (auto-removes on exit) so repeated runs either fail loudly (name conflict) or clean up after themselves instead of quietly accumulating.

> 🧠 **Remember:** `docker run` always creates a new container — five runs means five containers on disk, not one being reused. `--rm` and `--name` prevent this from accumulating silently.

> 🎯 **What this tests:** Whether you understand container identity well enough to reason about actual machine state, not just what the last command did — and whether you know the practical habit (`--rm`, `--name`) that avoids the problem entirely.

---

### 3. A teammate says "let's just use a bigger VM instead of containers, it's simpler." How do you respond?

**Answer:**
Start by agreeing with the part that's actually true: a VM *is* conceptually simpler — fewer moving parts, a well-understood operational model, no need to learn Docker or Kubernetes. That's a legitimate point, not something to dismiss.

Then reframe it as a trade-off question rather than a right/wrong one. A bigger VM doesn't solve the underlying problem on its own — if the team deploys straight onto a shared VM without a strict, repeatable process, the same "works on my machine" drift can creep back in at the VM level. It also doesn't give you cheap per-service isolation: spinning up a full VM per service costs minutes to boot, real memory/OS-licensing overhead, and doesn't scale down or up quickly.

The right move in an interview (and in real life) is to ask a follow-up question instead of arguing: *"What are we actually trying to avoid — complexity, or losing isolation between services?"* If it's a genuinely small, single-team, low-scale system with no need to scale services independently, one or two well-managed VMs might really be the pragmatic choice — Docker isn't a mandatory default. But if there's any expectation of scaling multiple services independently, deploying frequently, or running in a shared cluster, containers give a far better cost-to-isolation ratio.

> 🧠 **Remember:** Don't defend containers on reflex — ask what they're actually trying to avoid: complexity, or lost isolation.

> 🎯 **What this tests:** Whether you're dogmatic ("containers are always better") or you reason from trade-offs and ask the right clarifying question first. Interviewers are listening for judgment, not a rehearsed pro-container speech.

---

### 4. During a production incident, a teammate suggests: "Let's just exec into the running container, tweak the config, and restart the process — we'll fix the Dockerfile properly later." How do you respond?

**Answer:**
Push back on the shortcut, but by explaining the actual failure mode, not just "we don't do that." A container's entire value proposition is that the *image* is the single source of truth — what's running is supposed to be an exact, reproducible instance of it. The moment you `exec` in and hand-edit a running container, that guarantee is gone: the fix now lives only inside that one container's writable layer, which disappears the instant it's restarted, rescheduled onto another node, or replaced during a routine rolling deploy or autoscale event — all things that happen on their own, outside your control. You end up with a fleet where some replicas silently have the fix and others don't, no audit trail of what changed, and a registry image that's now lying about what's actually running.

The right move under pressure is still to fix it through the image: patch the source, rebuild, push, redeploy. If the real objection is speed, that's a pipeline problem to solve — a fast-tracked emergency deploy path — not a reason to bypass the image. The one legitimate exception worth naming: `exec`-ing in *read-only*, purely to inspect logs or state to diagnose the problem faster, is fine. The line is diagnosis versus mutation, not "never touch a running container."

> 🧠 **Remember:** Fix the image, not the running container — anything patched live in the writable layer disappears on the next restart or reschedule.

> 🎯 **What this tests:** Whether you hold the immutable-infrastructure principle under real incident pressure, or cave to "just this once." A weak answer either agrees to the shortcut or refuses without explaining *why* the fix would vanish — a strong answer names the concrete failure mode and still offers a path to move fast safely.
