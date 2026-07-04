# 🎤 Docker Interview Questions — Troubleshooting

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

These test how you actually debug — not the theory, the triage process.

---

### 1. A container that ran fine in dev crashes immediately in production with no obvious code error. What's the first category of cause you'd investigate, and why?

**Answer:**
Before touching code, check the actual crash reason and exit code — `docker logs` / `kubectl logs`, plus `docker inspect` or `kubectl describe pod` for the exit code. That single number already narrows the search: `137` means the container was killed (commonly an out-of-memory kill under a cgroup limit), `143` means it received `SIGTERM`, and a plain `1` usually points back to an application-level error.

With that in hand, the categories to check, roughly in the order they cause this exact symptom in real production incidents:

1. **Missing or different config/secrets** — env vars, secrets, or config maps that exist in dev (often via defaults or `.env` files) but were never provisioned in production. This is the single most common cause of "works in dev, dies instantly in prod."
2. **Resource limits** — production almost always sets cgroup/Kubernetes memory and CPU limits that dev never had. If the app's real memory usage exceeds that limit, it gets OOM-killed immediately on startup (exit code `137`).
3. **Image drift** — production may be pulling a different tag or a `:latest` that changed underneath since it was last tested, rather than the exact image that was verified in dev or staging.
4. **Filesystem/permissions differences** — production containers are frequently run as a non-root user or with a read-only root filesystem for security, and the app silently assumed write access or a privileged port it had in dev.
5. **Networking/startup dependencies** — DNS, service discovery, or network policies that exist in production but not in dev, causing the app to fail immediately when it can't reach a database or dependent service at boot.

In practice, the first two — missing config/secrets and OOM kills from resource limits — account for the large majority of "crashes instantly in prod, no code error" incidents, so that's where a senior engineer looks before assuming the code itself is wrong.

> 🎯 **What this tests:** Whether you have an actual triage process (start from the exit code, work outward) instead of guessing at causes. Interviewers are also listening for whether you'd jump straight to blaming the code — a weaker answer starts there instead of ruling out environment differences first.
