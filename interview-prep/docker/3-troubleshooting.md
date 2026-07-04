# 🎤 Docker Interview Questions — Troubleshooting

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

These test how you actually debug — not the theory, the triage process.

---

### 1. `docker run` fails with "Cannot connect to the Docker daemon." What do you check, in order?

**Answer:**
This error means the client parsed your command fine — the failure is entirely on the daemon side, so the debugging path is a service-health check, not a Docker-syntax check. In order:

1. **Is the daemon actually running?** `systemctl status docker` (Linux). If it's not active, start it (`systemctl start docker`) and check `journalctl -u docker` for why it isn't — a corrupted state directory or a config error are the common causes of it refusing to start.
2. **Do you have permission to reach the socket?** The daemon listens on `/var/run/docker.sock`, owned by root and the `docker` group. If your user isn't in that group, every command fails with this exact error even though the daemon is perfectly healthy — `sudo usermod -aG docker $USER` (then re-login) is the fix, not reinstalling Docker.
3. **Is `DOCKER_HOST` pointed somewhere unreachable?** If that environment variable is set (common after working with a remote daemon or a VM-based setup), the client is trying to reach a different host or socket entirely than the one actually running locally.
4. **Disk or resource exhaustion.** A full disk can cause the daemon to become unresponsive or crash outright — `df -h` on the daemon's data root is worth a quick check before assuming it's a config problem.

> 🧠 **Remember:** "Cannot connect to the daemon" is a service/permissions problem, not a command-syntax problem — start with `systemctl status docker` and socket group membership before anything else.

> 🎯 **What this tests:** Whether you know this error means the *client* is fine and localize the fault correctly, instead of re-typing the same command hoping it resolves itself.

---

### 2. A container shows as `Exited` immediately after starting. What's your triage sequence?

**Answer:**
Work outward from the container itself before assuming anything about the environment:

1. **`docker ps -a`** — confirm it actually exited (rather than never having started) and note the exit code shown next to its status.
2. **Read the exit code first.** `0` means the main process finished and returned cleanly — often because the image's default command wasn't actually a long-running process (a classic first-timer mistake, not a bug). `137` means it was killed, most often an out-of-memory kill. `1` or other non-zero codes point to an application-level failure.
3. **`docker logs <container>`** — this alone explains the majority of immediate exits: a missing config value, a stack trace, a bind-address conflict.
4. **`docker inspect <container>`** — check the configured `Cmd`/`Entrypoint`, mounted volumes, and environment variables actually applied, in case the container never had what it needed to begin with.

This general sequence is the entry point; the next question goes deeper into the specific *categories* of root cause once you've reached this point and still need to explain *why*.

> 🧠 **Remember:** `docker ps -a` for the exit code, `docker logs` for the reason, `docker inspect` for the configuration — in that order, before touching the application code.

> 🎯 **What this tests:** Whether you have a repeatable, ordered triage habit for the single most common Docker support question, rather than jumping straight to guessing.

---

### 3. A container that ran fine in dev crashes immediately in production with no obvious code error. What's the first category of cause you'd investigate, and why?

**Answer:**
Before touching code, check the actual crash reason and exit code — `docker logs` / `kubectl logs`, plus `docker inspect` or `kubectl describe pod` for the exit code. That single number already narrows the search: `137` means the container was killed (commonly an out-of-memory kill under a cgroup limit), `143` means it received `SIGTERM`, and a plain `1` usually points back to an application-level error.

With that in hand, the categories to check, roughly in the order they cause this exact symptom in real production incidents:

1. **Missing or different config/secrets** — env vars, secrets, or config maps that exist in dev (often via defaults or `.env` files) but were never provisioned in production. This is the single most common cause of "works in dev, dies instantly in prod."
2. **Resource limits** — production almost always sets cgroup/Kubernetes memory and CPU limits that dev never had. If the app's real memory usage exceeds that limit, it gets OOM-killed immediately on startup (exit code `137`).
3. **Image drift** — production may be pulling a different tag or a `:latest` that changed underneath since it was last tested, rather than the exact image that was verified in dev or staging.
4. **Filesystem/permissions differences** — production containers are frequently run as a non-root user or with a read-only root filesystem for security, and the app silently assumed write access or a privileged port it had in dev.
5. **Networking/startup dependencies** — DNS, service discovery, or network policies that exist in production but not in dev, causing the app to fail immediately when it can't reach a database or dependent service at boot.

In practice, the first two — missing config/secrets and OOM kills from resource limits — account for the large majority of "crashes instantly in prod, no code error" incidents, so that's where a senior engineer looks before assuming the code itself is wrong.

> 🧠 **Remember:** Start from the exit code, not the code — 137 means OOM-killed, 143 means SIGTERM, and most "works in dev, dies in prod" bugs are missing config or a resource limit, not an application bug.

> 🎯 **What this tests:** Whether you have an actual triage process (start from the exit code, work outward) instead of guessing at causes. Interviewers are also listening for whether you'd jump straight to blaming the code — a weaker answer starts there instead of ruling out environment differences first.
