# 🐳 Docker

> Containers, from first principles to production — part of the [Software Engineering Handbook](../README.md).

This section documents Docker the way it should be learned: understand the problem it solves before touching a single command, then build up from images and containers to networking, Compose, security, optimization, and real production usage.

Modules are added here only once their content is actually written and reviewed — not upfront as empty placeholders. This index grows alongside the content.

---

## ✅ Prerequisites

Install steps go stale fastest and the official docs already cover them precisely, so this is intentionally a checklist, not a chapter:

- Install **Docker Desktop** (macOS/Windows) or **Docker Engine** (Linux/EC2) — see the [official install docs](https://docs.docker.com/engine/install/).
- Verify it worked:

```bash
docker version
docker info
docker run hello-world
```

If `hello-world` runs successfully, you're ready for [01 — Introduction to Docker](./01-introduction/README.md).

---

## 📖 Learning Path

| # | Module | Status |
|---|--------|--------|
| 01 | [Introduction to Docker](./01-introduction/README.md) | ✅ Complete |
| 02 | [Docker Fundamentals](./02-fundamentals/README.md) | ✅ Complete |
| 03 | [Docker CLI](./03-docker-cli/README.md) | ✅ Complete |
| 04 | Docker Images | 🔜 Next up |
| 05 | Docker Containers | ⏳ Planned |
| 06 | Dockerfile & Image Building | ⏳ Planned |
| 07 | Docker Volumes | ⏳ Planned |
| 08 | Docker Networking | ⏳ Planned |
| 09 | Docker Compose | ⏳ Planned |
| 10 | Docker Registry & Image Distribution | ⏳ Planned |
| 11 | Docker Security | ⏳ Planned |
| 12 | Docker Optimization | ⏳ Planned |
| 13 | Docker Debugging | ⏳ Planned |
| 14 | Docker Troubleshooting Guide | ⏳ Planned |
| 15 | Production Best Practices | ⏳ Planned |
| 16 | CI/CD with Docker | ⏳ Planned |
| 17 | Real-World Docker Demos | ⏳ Planned |
| 18 | Docker Cheatsheet | ⏳ Planned |

Modules are added to this table with real links only once they're written and reviewed — the rows above are the plan, not a promise of order. Interview questions aren't a separate module: every chapter keeps its own bare questions, answered in full over in [Interview Prep → Docker](../interview-prep/docker/README.md).

---

## 🎯 What Each Module Covers

Every module in this section follows the same standard: the problem before the solution, a real-life analogy, internal working explained (not just commands), production best practices, common mistakes, and interview questions across beginner to advanced levels.

Full, answered versions of every module's interview questions live in [Interview Prep → Docker](../interview-prep/docker/README.md) — the chapters themselves keep just the questions, for self-testing before you check the answers.
