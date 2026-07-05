# 🖥️ Computer Systems — Troubleshooting

> Part of the [Software Engineering Handbook](../../README.md) → [Interview Prep](../README.md) → [Computer Systems](./README.md)

---

### 1. A computer powers on, fans spin, but the monitor shows no display at all. Which components would you check first, and why?

**Answer:**
"Fans spin" tells you power delivery and the motherboard are at least partially alive — so the investigation should move down the same journey the chapter walks through, checking each handoff for where the chain actually breaks:

1. **Physical display connection first.** Before suspecting any internal component, rule out the cheapest explanation: the monitor cable, the monitor's input source selection, or the monitor's own power. This isn't glamorous, but it's the single most common real-world cause and costs seconds to check.
2. **GPU seating and connection second.** If the GPU is a discrete card, confirm it's fully seated in its slot and receiving any required separate power connector. A GPU that isn't making good contact can leave the whole system "on" (fans spinning, motherboard powered) while producing no video signal at all.
3. **RAM seating third.** Many motherboards will POST (power-on self-test) with no display output at all — no beep, no error, just silence — if RAM isn't seated correctly. This looks identical to a dead GPU from the outside, which is why it comes right after.
4. **CPU and motherboard last.** If display cabling, GPU, and RAM all check out, suspect the CPU or motherboard itself — a failed CPU or a dead motherboard component can also produce "powers on but no display," but it's the least common cause and the most expensive to diagnose, so it's investigated last, not first.

The reasoning pattern matters more than the specific answer: work backward along the same Keyboard → Motherboard → CPU → RAM → GPU → Monitor journey, checking the cheapest, most likely failure points before the expensive, unlikely ones.

> 🧠 **Remember:** "Powers on but no display" is a GPU/RAM/connection problem 90% of the time, not a dead CPU or motherboard — check cheap and likely before expensive and rare.
>
> 🎯 **What this tests:** Whether you troubleshoot hardware the way an experienced engineer does — systematically, cheapest-and-most-likely first — instead of guessing randomly or jumping straight to "replace the motherboard."

---

**Source:** [How a Computer Actually Works](../../computer-science-fundamentals/01-computer-systems/01-how-computers-work/README.md)
