# 🎤 Docker Interview Questions — Production

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

These test whether you can actually run Docker safely and reliably in production — not just build and run a container locally.

---

### 1. How would you keep a Docker image small and secure for production, and why does image size actually matter operationally?

**Answer:**
Start with **multi-stage builds**: compile or build the app in one stage with all the heavy build tools installed, then copy only the final artifact into a slim runtime stage (`alpine`, `distroless`, or a minimal base) that never sees the compilers or package managers used to build it. Pin dependency versions instead of floating on `latest`, remove build-only packages from the final image, and use a `.dockerignore` so the build context (and by extension, the image) doesn't pick up local junk, secrets, or `.git` history. Ordering Dockerfile instructions so rarely-changing layers (dependency installs) come before frequently-changing ones (app code) also keeps rebuilds fast via layer caching — a build-speed win, not just an image-size one.

Why it matters beyond "smaller is tidier": a smaller image means fewer installed packages, which means a smaller CVE attack surface — you can't have a vulnerability in a library that was never in the image. It also means faster pulls and faster container starts, which matters directly at scale: a 2GB image pulled across a few hundred nodes during a rollout is a materially slower, more expensive deploy than a 100MB one.

> 🧠 **Remember:** Smaller image, smaller attack surface, faster rollout — multi-stage builds get you all three at once.

> 🎯 **What this tests:** Whether the candidate connects image size to both cost *and* security, instead of treating "use alpine" as the whole answer.

---

### 2. In production, how do you make sure a crashed or unhealthy container gets replaced automatically — and what's Docker's job versus the orchestrator's job in that?

**Answer:**
Plain Docker gives you host-local self-healing: a `--restart` policy (`on-failure`, `always`, `unless-stopped`) restarts a crashed container on the same host, and a `HEALTHCHECK` instruction in the Dockerfile defines what "healthy" actually means for that container, so `docker ps` can report an `unhealthy` status instead of just "running." That's genuinely useful, but it stops at the boundary of one host — if the host itself goes down, Docker can't reschedule that container anywhere else.

That's the orchestrator's job. Kubernetes uses the same idea — liveness and readiness probes, conceptually the same as a `HEALTHCHECK` — but enforces it cluster-wide: it reschedules failed containers onto healthy nodes, performs rolling restarts without downtime, and continuously reconciles the desired state across the whole fleet, not just one machine.

> 🧠 **Remember:** Docker's restart policy is host-local self-healing; the orchestrator's probes give you cluster-wide self-healing.

> 🎯 **What this tests:** Whether the candidate understands exactly where Docker's responsibility ends and the orchestrator's begins — a very commonly conflated boundary in interviews.

---

### 3. How should secrets — API keys, database passwords — be handled for containers in production? What's wrong with baking them into the image or passing them as plain environment variables?

**Answer:**
Baking a secret into the image (`ENV` in the Dockerfile, or `COPY`-ing a credentials file in) is close to a permanent leak: it's embedded in the image's layer history, present in every copy pulled from the registry, and readable by anyone who can pull that image — and rotating it means rebuilding and redistributing the image entirely.

Plain environment variables passed at runtime (`docker run -e`) are a step up, but still not production-grade for genuinely sensitive values: they're visible via `docker inspect`, readable from the container's process environment, and easy to leak accidentally into logs or error reports.

Production-grade options actually used at scale: mounted secret files — Kubernetes Secrets or Docker Swarm secrets mounted as files/volumes rather than baked into the image or config, so they can be rotated independently of a deploy — or a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault) that the app fetches from at startup, so nothing sensitive lives in the image, the orchestrator config, or source control at all. A related habit worth stating explicitly: `.env` files should only ever hold non-secret local defaults, and real credentials should never be committed to version control, even in a private repo.

> 🧠 **Remember:** Never bake secrets into the image or pass them as plain env vars — mount them as files or fetch them from a secrets manager at startup.

> 🎯 **What this tests:** Security judgment. A weak answer stops at "use environment variables." A strong one explains *why* image-baked and plain env-var secrets are risky, and names an actual secret-management mechanism used in real production systems.

---

### 4. Where should container logs go in production, and why is writing logs to a file inside the container the wrong approach?

**Answer:**
Containers should write logs to **stdout/stderr**, not to a file inside the container's filesystem. Docker's logging driver captures whatever is written to stdout/stderr and ships it wherever it's configured to go — locally to `json-file` for a single host, or forwarded to a centralized system like CloudWatch Logs, Fluentd, or Loki in production.

Writing logs to a file inside the container breaks this in two ways. First, containers are meant to be disposable — they get killed and replaced constantly in production, by design (scaling events, rolling deploys, node failures), and anything written only inside a container's writable layer disappears the moment that container is gone. Second, it doesn't scale operationally: with many replicas of the same service, logs trapped inside individual containers mean someone has to `exec` into each one separately, instead of reading one aggregated stream across all of them.

Treating stdout/stderr as the log destination — and letting the platform handle aggregation from there — is precisely what makes centralized log aggregation, and observability more generally, actually possible at scale.

> 🧠 **Remember:** Logs go to stdout/stderr, never to a file inside the container — containers are disposable, the log stream shouldn't be.

> 🎯 **What this tests:** Whether the candidate connects the "containers are disposable and stateless" principle to a concrete operational consequence (logging), rather than treating logging as a separate, unrelated concern.
