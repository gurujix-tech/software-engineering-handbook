# 🖥️ Computer Systems — Core Concepts

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Computer Systems](./README.md)

---

### 1. What are the main hardware components inside a computer, and what is each one responsible for?

**Answer:**
A computer is a team of specialized components, not one single "brain." The main ones are:

- **Keyboard (or any input device)** — detects a physical event (a key press) and converts it into a digital signal (a scan code). It doesn't know what letter was typed, only which switch closed.
- **Motherboard** — the physical wiring system. It doesn't think or decide anything; it's the circuit board every other component is connected to, providing the electrical pathways for signals to travel between components.
- **CPU (Central Processing Unit)** — the decision-maker. It executes instructions extremely fast, one after another, to figure out what a piece of incoming data means and what should happen next.
- **RAM (Random Access Memory)** — fast, temporary working memory. The CPU can only hold a tiny amount of information internally, so it constantly uses RAM as scratch space while it works.
- **GPU (Graphics Processing Unit)** — a specialized processor for massively parallel work. Originally built to render pixels, it's equally suited to any task that repeats the same simple calculation across huge amounts of data (which is why it also powers AI training).
- **Monitor** — converts the final image data it receives into visible light, pixel by pixel.

> 🧠 **Remember:** Every component has exactly one specialized job — the relay-race mental model. Remove any one of them, and the whole journey breaks down.
>
> 🎯 **What this tests:** Whether you understand hardware as a system of specialists rather than being able to only name the CPU and call it "the computer."

---

### 2. What is the difference between the CPU and the GPU?

**Answer:**
The CPU is a general-purpose decision-maker — it's good at handling many different kinds of tasks, one after another, very fast. It doesn't have built-in knowledge of "keyboards" or "screens"; it just executes whatever instructions software gives it.

The GPU is a specialist — it's built to do one repetitive kind of calculation (like "color this pixel") across millions of pieces of data at the same time, in parallel. The CPU decides *what* should happen; the GPU is handed a narrow, well-defined job and executes it at massive scale.

That's also why the same GPU hardware strength shows up in two very different places: rendering millions of pixels per frame in a game, and running millions of matrix calculations per step when training an AI model. Same strength — do one simple thing across a lot of data at once — two very different use cases.

> 🧠 **Remember:** CPU = few tasks, sequentially, very flexibly. GPU = one task, in massive parallel, very fast.
>
> 🎯 **What this tests:** Whether you can explain the CPU/GPU split by *workload shape* rather than just reciting "CPU is for general stuff, GPU is for graphics."

---

### 3. Why does a computer need RAM if it already has permanent storage (like an SSD)?

**Answer:**
Because permanent storage and RAM are optimized for opposite things. An SSD is optimized to keep data safe even when the power is off — it's permanent, but that durability makes it much slower to read and write compared to RAM. RAM is optimized purely for speed: it can be read and written to almost instantly, but it trades away permanence to get that speed — it forgets everything the moment the computer loses power.

While the CPU is actively working (converting a scan code into a character, tracking which application is focused, deciding where a letter should be drawn), it needs a workspace that can keep up with how fast it operates. If the CPU had to read and write every intermediate piece of data to an SSD, it would spend most of its time waiting instead of computing. RAM exists so the CPU always has a fast, temporary place to hold what it's actively working on.

> 🧠 **Remember:** RAM trades permanence for speed; storage trades speed for permanence. A computer needs both because it needs to *work* fast and *remember* safely — those are different jobs.
>
> 🎯 **What this tests:** Whether you understand the fundamental speed-vs-permanence trade-off in memory hierarchy, which is the foundation for later topics like caching, paging, and swap.

---

### 4. What role does the motherboard play in how components communicate with each other?

**Answer:**
The motherboard is the physical connection layer — the large circuit board that every other component (CPU, RAM, GPU, storage, input devices) is physically wired into. It doesn't process, store, or decide anything itself. Its only job is to provide the electrical pathways that let a signal travel from one component to another.

Without the motherboard, none of the components could communicate at all, because there would literally be no physical pathway between them — a CPU sitting next to a stick of RAM with no motherboard connecting them is just two disconnected pieces of silicon. The motherboard is what turns a pile of separate components into one working system.

> 🧠 **Remember:** The motherboard is the roads, not a driver — it doesn't decide where anything goes, it just makes sure a route exists.
>
> 🎯 **What this tests:** Whether you can distinguish "components that process/decide" (CPU, GPU) from "the infrastructure that connects them" (motherboard) — a distinction that maps directly onto network/infrastructure thinking later in your career.

---

### 5. Walk through, at a hardware level, everything that happens between a key press and a character appearing on a monitor.

**Answer:**

This is a step-flow question — the interviewer wants to see that you can narrate a full system end-to-end, in the right order, without skipping a component.

1. **Keyboard detects the press.** Every key sits above a switch on a grid. Pressing it completes a circuit at a known row/column position. The keyboard's own small controller converts that physical position into a digital scan code — it does not yet know this represents the letter "A."
2. **Signal travels to the motherboard.** The scan code moves as an electrical signal over a cable (or internal connection on a laptop) to the motherboard — the circuit board providing the physical pathway to the rest of the system.
3. **The Operating System receives the event.** The OS is the layer sitting between hardware and every decision made from this point forward — it receives the key event first and hands it off appropriately. (Its internals are a separate topic — for this answer, the important part is that it sits here, coordinating.)
4. **The CPU executes instructions to interpret it.** The CPU receives the scan code as data and runs the instructions needed to map it to the character "A" and decide what should happen as a result (e.g., which application should receive it).
5. **RAM holds the working state.** While the CPU is doing this, it uses RAM as fast temporary storage — for the character itself, which application is focused, and where the letter should be drawn. The CPU only holds a tiny amount of data internally, so RAM is doing the heavy lifting of "remembering" mid-task.
6. **The GPU renders it.** Once the CPU has decided *what* needs to appear (character "A," specific position, specific font), it hands that decision to the GPU, which calculates the actual pixels needed to draw it — a job it can do across millions of pixels in parallel.
7. **The monitor displays it.** The GPU sends the finished pixel data to the monitor, which turns that signal into visible light — activating the right pixels, in the right colors, at the right positions.

![The journey of a key press: Press "A" flows through Keyboard, Motherboard, Operating System, CPU, RAM, and GPU before the Monitor displays "A"](../../assets/diagrams/computer-science-fundamentals/computer-systems/how-computers-actually-work.png)

> 🧠 **Remember:** The order never changes — Keyboard → Motherboard → OS → CPU → RAM → GPU → Monitor. If you can name the order under pressure, you can answer any variant of this question (mouse click, network packet arriving, etc.) using the same shape.
>
> 🎯 **What this tests:** Whether you actually understand the full system as one continuous pipeline, or only know isolated facts about each component. This is the single best "do they really get it" question in this topic.

---

**Source:** [How a Computer Actually Works](../../computer-science-fundamentals/01-computer-systems/01-how-computers-work/README.md)
