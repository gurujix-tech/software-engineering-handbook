# 🐳 Docker CLI

> Part of the [Software Engineering Handbook](../../README.md) → [Docker](../README.md)

---

## 📋 Chapter Info

| | |
|---|---|
| **Module** | 03 — Docker CLI |
| **Prerequisites** | [02 — Docker Fundamentals](../02-fundamentals/README.md) |
| **Estimated Reading Time** | 12–15 minutes |
| **Difficulty** | Beginner |

---

## 🧭 Overview

You know the theory now: client, daemon, registry, and a container's four lifecycle states. This chapter is where that theory becomes muscle memory — the small, fixed set of commands you'll actually type every single day, each one just a way of moving a container (or an image) between the states you already understand.

Nothing here is new architecture. It's vocabulary.

---

## 🎯 Learning Objectives

By the end of this chapter, you should be able to:

- Run, inspect, stop, and remove a container using only the commands covered here
- Explain the difference between `docker run`, `docker start`, and `docker exec` without hesitating
- Read `docker ps -a` output and know exactly what state each container is in and why
- Diagnose a container that exited immediately using `docker logs` and `docker inspect`

---

## 🧠 Mental Model

**The Docker CLI is a small, closed vocabulary mapped directly onto the lifecycle you already know.**

There is no command that does something outside of "operate on an image" or "move a container between lifecycle states." Once you stop treating each command as its own thing to memorize and start asking "which lifecycle transition is this," the whole command set collapses from "dozens of things to remember" into "four or five patterns applied to two kinds of objects."

> 💡 **Key Insight:** Every command below is either an *image* command (pull, images, rmi) or a *container* command (run, ps, logs, exec, inspect, stop, start, rm). Sort new commands into one of those two buckets and they stop feeling random.

---

## ❗ The Problem: Copy-Pasted Commands, No Mental Map

Most people's first contact with the Docker CLI is copy-pasting a command from a tutorial without understanding it — and it works, which is almost worse, because now there's no reason to go back and actually learn it. Then the small variations show up and nothing makes sense: `docker run` was copy-pasted and the terminal never returned control, so `-d` gets added because "that's what StackOverflow said," with no idea it means "detached, don't hold my terminal hostage." `docker ps` shows nothing, and it's unclear whether that means no containers exist or the container just isn't *running* (it's `docker ps -a` that would have shown it, sitting there `Exited`). A container gets removed, `docker rmi` is run to clean up the image behind it, and it fails with a dependency error that seems to come from nowhere.

None of these are Docker being difficult. They're the predictable result of typing commands without a lifecycle model — which is exactly what the previous chapter gave you, and exactly what this chapter now attaches real commands to.

---

## 🍳 Real-Life Analogy: The Restaurant, Continued

Picking up the restaurant analogy from the last chapter, applied to specific orders:

- **`docker pull`** — the kitchen calling the supplier ahead of time to stock an ingredient, before anyone's even ordered.
- **`docker run`** — placing a full order: "make this dish and bring it to my table now" (create the container *and* start it).
- **`docker ps`** — asking "which tables currently have food being actively served?"
- **`docker ps -a`** — asking "show me every table today, served or not, including the ones that already finished and left their plates."
- **`docker logs`** — asking the kitchen what actually happened while a specific dish was being made.
- **`docker exec`** — stepping into the kitchen yourself, mid-cook, to check on a dish without stopping it.
- **`docker stop`** — asking the kitchen to pause a dish — the plate stays on the table, it's just not being actively cooked anymore.
- **`docker rm`** — actually clearing the plate off the table.
- **`docker rmi`** — telling the supplier you no longer need that recipe on file — which the kitchen will refuse if a plate made from it is still sitting on a table.

---

## 🖼️ Hero Illustration

- **Purpose:** Map the CLI commands from this chapter directly onto the lifecycle-state diagram from the previous chapter, so the two chapters visually connect.
- **Learning Objective:** Every container command is a labeled arrow between two lifecycle states — nothing more.
- **Illustration Description:** Reuse the four-state horizontal diagram (Created → Running → Stopped → Removed) from the Fundamentals chapter, but now label each arrow with its actual command instead of a generic verb: `docker create` (into Created), `docker start` (Created→Running, and Stopped→Running), `docker run` (a curved arrow shown going straight from outside the diagram into Running, labeled "create + start in one step"), `docker stop` (Running→Stopped), `docker rm` (Stopped→Removed). Add small side-notes off to the side (not inline on the arrows) for `docker ps` / `docker ps -a` / `docker logs` / `docker exec` / `docker inspect` as "commands that observe state, without changing it."
- **Placement:** Directly below this section, before "Your Daily Docker Commands."
- **Image Generation Notes:** Flat 2D vector style, same color palette as the Fundamentals lifecycle diagram for consistency (gray/green/amber/red per state), commands in a monospace-style label font. Bottom-right attribution: "ⓘ Vamsi Chunduru | Gurujix".

![Hero Illustration: Docker CLI commands mapped onto container lifecycle states](../../assets/diagrams/docker/docker-cli-lifecycle-map.png)

---

## 📖 Your Daily Docker Commands

Split into the two buckets from the mental model above.

**Image commands** — operate on the blueprint, not a running instance:

| Command | What it does |
|---|---|
| `docker pull <image>` | Fetches an image from a registry into your local cache. |
| `docker images` | Lists every image currently cached locally. |
| `docker rmi <image>` | Removes a locally cached image — fails if a container still depends on it. |

**Container commands** — create, observe, or move a container between lifecycle states:

| Command | What it does |
|---|---|
| `docker run <image>` | Creates *and* starts a container in one step — the one you'll use almost always. |
| `docker run -d --name <name> <image>` | Same, but detached (`-d`, doesn't hold your terminal) and given a human-readable name instead of a random one. |
| `docker ps` | Lists only *running* containers. |
| `docker ps -a` | Lists *every* container — running, stopped, everything. |
| `docker logs <name>` | Shows the stdout/stderr the container's process has produced. |
| `docker exec -it <name> sh` | Opens an interactive shell inside an already-running container. |
| `docker inspect <name>` | Dumps the full config Docker actually applied — env vars, mounts, ports, restart policy. |
| `docker stop <name>` | Sends a stop signal — the container moves to Stopped, but still exists. |
| `docker start <name>` | Restarts a previously-stopped container — same container, same ID. |
| `docker rm <name>` | Permanently deletes a stopped container. |

> 🎯 **Quick Check:** If you're ever unsure which bucket a new command belongs to, ask: "does this need an image, or does it need a container that already exists?" That question alone gets you most of the way to guessing correctly.

---

## ⚙️ How It Works: A Full Lifecycle Walkthrough

Here's the sequence above, run for real, end to end:

```bash
docker pull nginx                    # fetch the image ahead of time (optional — run would do this too)
docker images                        # confirm it's now cached locally

docker run -d --name web nginx       # create + start, detached, named "web"
docker ps                            # "web" shows up — it's running

docker logs web                      # see nginx's startup output
docker exec -it web sh               # step inside the running container to look around
docker inspect web                   # see the full config Docker actually applied

docker stop web                      # container moves to Stopped — it still exists
docker ps                            # "web" is gone from this list...
docker ps -a                         # ...but it's still here, with status "Exited"

docker start web                     # same container, same ID, running again
docker rm web                        # (after stopping again) permanently delete it

docker rmi nginx                     # remove the image — fails if any container still references it
```

> 🧠 Notice `docker ps` and `docker ps -a` disagree right after `stop` — that's not a bug, it's the entire point of the two commands existing separately. `ps` answers "what's running right now," `ps -a` answers "what exists at all."

---

## ❌ Common Misconceptions

- **"`docker start` and `docker run` do the same thing."** `run` always creates a brand-new container from an image. `start` only works on a container that already exists (usually one that was previously stopped) — it does not create anything.
- **"`docker exec` is how you start a container."** `exec` only works against a container that's *already running* — it opens a second process inside it. It cannot start a stopped container; that's what `docker start` is for.
- **"An empty `docker ps` means no containers exist."** It means none are currently *running*. `docker ps -a` is the command that answers "do any exist at all."

---

## ✅ Best Practices

- Always name containers with `--name` in anything beyond a one-off test — `docker logs web` is far easier to come back to than `docker logs a3f9c2e1b7d4`.
- Default to `-d` for anything long-running (a web server, a database) — an attached container ties up your terminal and dies if you close it.
- Reach for `docker ps -a` by default when troubleshooting "where did my container go" — it is almost always still there, just stopped.
- Check `docker logs` before `docker inspect` when something's wrong — logs tell you *what* happened; inspect tells you *how it was configured* to happen.

---

## 🚨 Common Mistakes

- **Forgetting `-a` on `docker ps`** and concluding a container "disappeared" when it's actually sitting there, stopped.
- **Running `docker rmi` on an image still in use** and being confused by the dependency error instead of first checking `docker ps -a` for containers built from it.
- **Running `docker run` repeatedly while debugging** instead of naming the container once and reusing `docker start` — this leaves behind a pile of duplicate stopped containers from the same image.
- **Never running `docker version`** after an install or upgrade — a client/server version mismatch is a real source of confusing, hard-to-google errors.

---

## 🎤 Interview Questions

**Beginner**

- What is the difference between `docker run`, `docker start`, and `docker exec`?
- What's the difference between `docker ps` and `docker ps -a`?

**Intermediate**

- Why does the first `docker run` of a given image behave differently from every run after it?

**Scenario**

- A container you stopped a week ago doesn't show up in `docker ps`. A teammate says it "disappeared." Walk through how you'd show them it didn't.
- `docker rmi` fails on an image you're sure isn't being used. What do you check?

> 📦 **In Practice:** Full, interview-ready answers to these — plus more advanced and production-scenario questions — are in [Interview Prep → Docker](../../interview-prep/docker/README.md).

---

## 📝 Summary

The Docker CLI isn't a long list to memorize — it's a small set of image commands (`pull`, `images`, `rmi`) and container commands (`run`, `ps`, `logs`, `exec`, `inspect`, `stop`, `start`, `rm`), each one a direct, predictable move against the lifecycle model from the previous chapter. Once that mapping clicks, new commands you encounter later mostly explain themselves.

---

## 🔑 Key Takeaways

- Every command is either an image command or a container command — sort new ones into those two buckets.
- `docker run` = `create` + `start` in one step; it's the default you'll reach for almost always.
- `docker ps` shows running containers only; `docker ps -a` shows everything, including stopped ones.
- `docker exec` requires an already-running container; it cannot start a stopped one.
- `docker logs` explains *what happened*; `docker inspect` explains *how it was configured*.

---

## ✅ Self Assessment

- [ ] I can explain the difference between `docker run`, `docker start`, and `docker exec` without hesitating.
- [ ] I know why `docker ps` and `docker ps -a` can disagree right after a `docker stop`.
- [ ] I can walk through diagnosing a container that exited immediately using `logs` and `inspect`.
- [ ] I can explain why `docker rmi` sometimes fails and what to check first.

---

## 📚 References

- [Docker CLI reference — official docs](https://docs.docker.com/reference/cli/docker/)
- [`docker run` reference — official docs](https://docs.docker.com/reference/cli/docker/container/run/)

---

## 🔮 What's Next?

You can now operate Docker day-to-day with confidence. The next chapter, **Docker Images**, goes one level deeper into the object you've only used so far — what a layer actually is, how layers are cached and shared, and why image size and structure matter well beyond "does it run."

---

## 🔗 Chapter Navigation

⬅️ **Previous:** [02 — Docker Fundamentals](../02-fundamentals/README.md)
➡️ **Next:** Docker Images — Coming soon
