# ⚙️ VLSI: 8-Bit Ripple-Carry Adder/Subtractor (with Animations)

**ECE571 — VLSI System Design (Spring 2024)**  
This repo contains RTL/schematics notes, waveforms, and **interactive visualizations** for an 8-bit ripple-carry adder/subtractor (plus D-FF, XOR for subtract, and a simple CLA reference). Includes timing/overflow views and a 16-bit cascade demo.

> Explore the animations:  
> - 🔗 [`viz/ripple-carry.html`](./viz/ripple-carry.html) – carry ripple animation (add/sub)  
> - 🔗 [`viz/timing-diagram.html`](./viz/timing-diagram.html) – animated timing & overflow  
> - 🔗 [`viz/cla-vs-rca.html`](./viz/cla-vs-rca.html) – CLA vs RCA visual delay comparison

---

## 🧭 Overview
- **Function:** 8-bit adder/subtractor with mode control, Cin/Cout, and cascade to 16-bit.
- **Subtract path:** XOR-invert B, add with Cin=1 → two’s-complement subtraction.
- **Blocks:** Full Adder, D-FF (for stable overflow capture), XOR gates, optional 4-bit CLA slice.
- **Why ripple-carry?** Simple, compact, cost-friendly; tradeoff is carry-prop delay.

---

## 🧩 Repo Layout


---

## 🧠 Concepts
- **Ripple Carry:** chain of full adders; each stage waits on previous **Cout → Cin**.
- **Subtract:** `B' = B ⊕ mode`, `Cin = mode`. When `mode=1`, perform `A + (~B + 1)`.
- **Overflow:** for signed add/sub, `OF = Cout[MSB] ⊕ Cint[MSB]`. D-FF can stabilize readout.
- **Cascade:** connect lower-byte `Cout` to upper-byte `Cin` for 16-bit ops.

---

## 🧪 What to Try in the Animations
- Toggle **ADD/SUB** and watch the **carry ripple** direction and speed.
- Flip input bits and observe **critical chain** (worst-case all 1→0 or 0→1 sequences).
- Compare **CLA vs RCA** to see how **generate/propagate** short-cuts the path.
- Trigger cases that set **overflow** and see it latched (D-FF).

---

## ▶️ Run Locally or on GitHub Pages
- Local: open files in `viz/` directly in your browser.
- GitHub Pages: **Settings → Pages → Deploy from branch** (root or `/docs`).  
  Put `/viz` under `/docs` (or keep root and link as above).

---

## 📚 References
- Full adder, XOR-based subtract path, D-FF stabilization, CLA slice, waveforms, and layout checks (DRC/LVS) were implemented and analyzed in the original lab work. See `paper/ece571_lab4_report.pdf` for schematics and waveforms.

