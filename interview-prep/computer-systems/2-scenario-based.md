# 🖥️ Computer Systems — Scenario-Based

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Computer Systems](./README.md)

---

### 1. A user says their application feels laggy specifically when typing. Which components would you suspect, and in what order would you investigate them?

**Answer:**
Start from the fact that the keyboard itself is almost never the actual cause — it only detects a physical event and hands off a scan code; there's very little for it to get "slow" at. The lag is happening somewhere downstream, in the chain the scan code travels through afterward. Investigate in this order:

1. **CPU load first.** Since the OS and CPU are the components that receive and act on every key event, check if the CPU is saturated (high overall utilization, or one thread pegged at 100%). If the CPU is busy with something else, it can't respond to input events promptly — this is the single most common real-world cause of "laggy typing."
2. **Memory pressure (RAM) second.** If the system is low on available RAM, the OS may be swapping data to disk to free up space, and disk is dramatically slower than RAM. That swapping activity can stall the process handling your keystrokes, producing exactly this kind of input lag.
3. **GPU/rendering third.** If the CPU and memory look healthy, check whether the lag correlates with what's on screen — a laggy text editor with heavy syntax highlighting, or a browser tab with an expensive animation, can bottleneck at the rendering step (GPU) even though the input path itself is fine.
4. **I/O last.** If the application is doing something on every keystroke that touches disk or network (e.g., autosave, live linting against a remote server), the input feels slow because it's waiting on I/O, not because any of the core hardware is actually struggling.

The key move in this question is *not* jumping to "add more RAM" or blaming the keyboard — it's naming the chain (CPU → RAM → GPU → I/O) and explaining why you'd check it in that order.

> 🧠 **Remember:** The keyboard is a red herring almost every time — the bottleneck is always somewhere in the CPU → RAM → GPU → I/O chain, and the chapter's relay-race mental model is exactly the checklist to walk through here.
>
> 🎯 **What this tests:** Whether you reason about performance problems as "which specialist in the chain is overloaded" rather than guessing at a random fix. This is the same instinct used for CPU-bound vs. memory-bound vs. I/O-bound triage in production systems later in your career.

---

**Source:** [How a Computer Actually Works](../../computer-science-fundamentals/01-computer-systems/01-how-computers-work/README.md)
