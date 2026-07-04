# 🎤 Docker Interview Questions — Core Concepts

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Docker](./README.md)

Straight conceptual questions — the ones that check whether you actually understand Docker, not just whether you've memorized commands. Ordered from foundational to senior-level.

---

### 1. What problem does Docker solve?

**Answer:**
Docker solves *environment inconsistency* — the "works on my machine" problem. Before Docker, an app's behavior depended on the exact OS version, installed libraries, and config of whatever machine it ran on, so the same code could work in dev and break in QA or production.

Docker packages the app together with its entire runtime environment — code, runtime, dependencies, config — into a single portable unit (an **image**) that behaves identically wherever it runs. It replaced two older approaches: detailed manual setup docs (fragile — one skipped step breaks everything) and full Virtual Machines (isolate well, but each one boots a full guest OS, paying real cost in memory, disk, and boot time). Docker gets VM-like isolation using two Linux kernel features — namespaces and cgroups — without the overhead of a separate OS per app.

> 🧠 **Remember:** Docker doesn't solve deployment — it solves environment consistency.

> 🎯 **What this tests:** Whether you understand *why* Docker exists, not just "it's containers." A strong answer connects environment consistency to the cost/isolation trade-off against VMs — not just "it's lightweight."

---

### 2. What's the difference between a Docker image and a Docker container?

**Answer:**
An **image** is a static, read-only blueprint — filesystem layers plus metadata (entrypoint, environment variables, exposed ports) built from a Dockerfile. It never runs by itself; it just sits in storage (locally or in a registry).

A **container** is a running (or stopped) *instance* created from an image, with its own writable layer on top, its own process, its own network namespace, and its own lifecycle. One image can produce many independent, simultaneously running containers — the same way one class definition produces many object instances.

> 🧠 **Remember:** One image, many containers — same relationship as a class and its objects.

> 🎯 **What this tests:** Precision. This is the single most common terminology mix-up in real teams — people say "I deployed a container" when they actually built and pushed an image. Interviewers use this question to see if you're exact with the vocabulary.

---

### 3. How is a container different from a VM at a mechanism level?

**Answer:**
A **VM** runs on a hypervisor that virtualizes hardware. Each VM boots its own full guest operating system — its own kernel, its own memory footprint, its own boot sequence — on top of that virtual hardware. This gives strong isolation, but it's heavy: gigabytes per image, seconds to minutes to boot.

A **container** has no hypervisor and no guest kernel. It shares the host's kernel. Isolation comes from two Linux kernel primitives instead:

- **Namespaces** — give the container its own *view* of the system (its own process tree, network interfaces, mounts, hostname), so it feels like a separate machine even though it's really just a process on the host.
- **Cgroups** — enforce resource *limits* (CPU, memory, I/O) so no single container can starve the others.

Because there's no separate OS to boot, containers start in milliseconds and images are typically megabytes, not gigabytes.

> 🧠 **Remember:** No hypervisor, no guest kernel — containers virtualize the OS, VMs virtualize the hardware.

> 🎯 **What this tests:** Mechanism-level understanding. "Containers are lighter" is not a complete answer — naming namespaces and cgroups specifically is what separates a memorized answer from real understanding.

---

### 4. What do namespaces and cgroups each do for a container?

**Answer:**
Split them cleanly — they solve two different problems:

- **Namespaces control visibility.** Each namespace type isolates one dimension of what a process can see: PID (its own process tree), network (its own interfaces/ports), mount (its own filesystem view), UTS (its own hostname), IPC, and user (its own UID/GID mapping). Together, they make a container believe it's on its own machine.
- **Cgroups (control groups) control consumption.** They cap and account for CPU shares, memory limits, and block I/O per container, and can throttle or OOM-kill a container that exceeds its budget. This is what prevents a "noisy neighbor" container from starving everything else on the host.

> 🧠 **Remember:** Namespaces decide what a container can see, cgroups decide what it can use.

> 🎯 **What this tests:** Depth beyond buzzwords — can you name concrete namespace types and explain what cgroups actually enforce, not just recite the two words.

---

### 5. Walk through what actually happens at the Linux kernel level when you run `docker run` — how does a container end up feeling like its own isolated machine?

**Answer:**
Nothing gets virtualized in the VM sense — there's no second kernel and no virtual hardware. A container ends up as a completely ordinary Linux process, just started with a deliberately restricted *view* of the one real kernel already running. Here's the flow, in order:

1. **Docker hands off the request.** The CLI passes `docker run` to the Docker daemon, which delegates the actual work to containerd.
2. **containerd manages the lifecycle.** It pulls the image if it isn't cached locally, then hands container creation to the low-level runtime, `runc`.
3. **`runc` calls `clone()` with namespace flags.** Flags like `CLONE_NEWPID`, `CLONE_NEWNET`, `CLONE_NEWNS`, `CLONE_NEWUTS`, `CLONE_NEWIPC`, `CLONE_NEWUSER`, `CLONE_NEWCGROUP` mean the process is born already isolated across seven dimensions — its own process tree, network stack, mounts, hostname, IPC, user/group mapping, and cgroup view.
4. **`pivot_root` swaps the filesystem.** The process's `/` is switched to the image's unpacked filesystem — the modern, safer version of `chroot`. Same kernel binary, entirely different filesystem view.
5. **OverlayFS mounts the image layers.** The image's read-only layers (`lowerdir`) stack under the container's writable layer (`upperdir`) into one merged view, so hundreds of containers can share the same base layers on disk via copy-on-write instead of duplicating them.
6. **A `veth` pair wires up networking.** A virtual ethernet pair connects the container's new network namespace to a bridge on the host (`docker0` by default) — its own IP, still routed through the host.
7. **cgroups and seccomp lock it down.** The process is placed into a cgroup that caps its CPU/memory/IO, and seccomp/capabilities filtering restricts which syscalls and privileges it can use at all — layered on top of the namespaces, not replacing them.
8. **The process starts** — as its own PID 1, believing it's alone on the machine.

Every one of those steps restricts *visibility or resource access*; none of them swap out the kernel itself. That's precisely why it's called **OS-level virtualization**, not virtualization in the hypervisor sense, and precisely why a container can never run a different kernel than its host (no Windows containers on a Linux host, no picking a different kernel version per container) — there's only ever one kernel underneath all of them.

![Diagram: the 8-step kernel-level flow of docker run, from the CLI handoff through containerd and runc to clone() namespaces, pivot_root, OverlayFS, veth networking, cgroups/seccomp, and PID 1 starting](../../assets/diagrams/docker/how-docker-runs-internal.png)

> 🧠 **Remember:** A container is just a regular Linux process — `clone()` with namespace flags, `pivot_root` for its filesystem, OverlayFS for layers, and cgroups for limits. There's no second kernel, ever.

> 🎯 **What this tests:** Whether you actually know the concrete kernel primitives — specific `clone()` flags, `pivot_root`, OverlayFS, `veth` — rather than re-reciting "namespaces and cgroups" as a memorized pair. This is usually the deepest filter question in a platform/DevOps interview, separating people who've read about containers from people who've had to debug one at the syscall level.

---

### 6. Explain the relationship between Docker, containerd, and the OCI spec.

**Answer:**
It's a layered stack, not competing tools:

- **Docker Engine** is the developer-facing platform — the CLI, Dockerfile builds, Compose, image management. Since the Moby restructuring, Docker doesn't handle low-level container lifecycle itself; it delegates that to containerd underneath.
- **containerd** is a lower-level, high-performance container runtime (a CNCF graduated project). Its job is narrow: pull images, manage container lifecycle (create/start/stop/delete), handle storage/snapshotting. It exposes an API that Kubernetes talks to directly via the Container Runtime Interface (CRI) — Kubernetes doesn't need Docker installed at all.
- **OCI (Open Container Initiative)** isn't a runtime — it's a specification. It defines the image format (OCI Image Spec) and the runtime contract (OCI Runtime Spec, implemented in practice by `runc`), so any OCI-compliant image runs on any OCI-compliant runtime.

Chained together: **Docker → containerd → runc → Linux kernel primitives (namespaces/cgroups)**, with OCI as the vendor-neutral contract every layer agrees to. That's exactly why an image built by Docker also runs directly under containerd or Kubernetes with no Docker installed on the node.

> 🧠 **Remember:** Docker → containerd → runc → kernel — OCI is the shared contract that keeps every layer swappable.

> 🎯 **What this tests:** Architectural understanding — this is a favorite at intermediate/senior DevOps interviews, especially framed as "why did Kubernetes remove dockershim in 1.24, and why did nothing actually break?"

---

### 7. If containers share the host kernel, what are the security implications, and how would you mitigate them?

**Answer:**
Because every container on a host shares one kernel, the isolation boundary is fundamentally weaker than a VM's:

- A kernel-level exploit or container-breakout CVE has one shared attack surface — a compromise can potentially affect the host and every other container on it, not just one isolated VM.
- Containers running as root, with `--privileged`, or with excessive Linux capabilities can reach host devices or namespaces they shouldn't.
- Without resource limits, a compromised or buggy container can consume host CPU/memory and degrade every co-located container (noisy neighbor / denial of service).

**Practical mitigations, in order of what actually matters in production:**

- Never use `--privileged` unless there's no alternative; drop all capabilities by default and add back only what's required (`--cap-drop=ALL`, then add specific ones).
- Run containers as a non-root user (`USER` in the Dockerfile) instead of the default root.
- Use read-only root filesystems where the app allows it.
- Enforce cgroup resource limits (`--memory`, `--cpus`, or Kubernetes resource requests/limits) on every container, not just the ones you suspect.
- Apply seccomp, AppArmor, or SELinux profiles to restrict available syscalls.
- For genuinely untrusted or multi-tenant workloads, add a stronger isolation layer: run containers inside VMs (the default on most managed cloud platforms already), or use a sandboxed runtime like **gVisor** or **Kata Containers**, which interpose an extra isolation layer between the container and the host kernel.
- Keep the host kernel and container runtime patched — most breakout CVEs get fixed quickly, but only if you're actually applying updates.

> 🧠 **Remember:** One shared kernel means one shared attack surface — drop capabilities, skip `--privileged`, and reach for gVisor/Kata for untrusted workloads.

> 🎯 **What this tests:** Security maturity. A weak answer stops at "don't run privileged containers." A strong senior-level answer gives a layered mitigation strategy and knows when to reach for gVisor/Kata for genuinely untrusted workloads.
