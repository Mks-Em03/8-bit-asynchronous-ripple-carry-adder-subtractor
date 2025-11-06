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
