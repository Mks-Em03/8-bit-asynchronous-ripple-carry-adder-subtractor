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

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>8-bit Ripple-Carry Adder/Subtractor — Animated</title>
<style>
  :root { --fa: 72px; --gap: 20px; --speed: 0.9s; font-family: Inter, system-ui, sans-serif; }
  body { margin: 24px; color: #111; background: #fff; }
  h1 { font-size: 22px; margin: 0 0 6px; }
  .sub { color:#555; margin-bottom: 18px; }
  .row { display: grid; grid-template-columns: repeat(8, var(--fa)); gap: var(--gap); align-items: center; }
  .stage { position: relative; height: var(--fa); border-radius: 14px; box-shadow: 0 1px 6px rgba(0,0,0,.12); }
  .stage.add { background: #f2f9ff; }
  .stage.sub { background: #fff5f2; }
  .bitlabel { position:absolute; top:-18px; left:6px; font: 12px/1.2 monospace; color:#666; }
  .pins { position:absolute; inset: 6px; display:grid; grid-template-rows: 1fr auto; }
  .io { display:flex; justify-content: space-between; font: 12px monospace; color:#333; }
  .carry { position:absolute; top: calc(50% - 4px); left: -12px; width: calc(100% + 24px); height: 8px; }
  .carry .bubble { position:absolute; top:0; width:8px; height:8px; border-radius:50%; background:#111; opacity:.12; }
  .carry.propagate .bubble { animation: move var(--speed) linear forwards; }
  .carry.blocked .bubble { opacity:.08; }
  @keyframes move {
    from { transform: translateX(0); opacity:.22; }
    to   { transform: translateX(calc(100% - 8px)); opacity:.85; }
  }
  .controls { display:flex; gap:12px; align-items:center; margin: 20px 0 10px; }
  button, input[type="number"], .toggle { font: 14px/1.1 system-ui; padding:8px 12px; border:1px solid #ddd; border-radius:10px; background:#fff; }
  button { cursor:pointer; }
  .toggle { display:inline-flex; gap:8px; align-items:center; cursor:pointer; }
  .toggle input { accent-color:#111; }
  .legend { font:12px monospace; color:#444; margin-top:10px; }
  .footer { margin-top:18px; font:12px; color:#666; }
</style>
</head>
<body>
  <h1>8-bit Ripple-Carry Adder/Subtractor</h1>
  <div class="sub">Visualize <b>carry propagation</b> across full-adder stages. Toggle add/sub, edit A and B, and watch the ripple.</div>

  <div class="controls">
    <label class="toggle"><input id="mode" type="checkbox" /> SUB (A − B)</label>
    <label>A: <input id="aval" type="number" min="0" max="255" value="173" /></label>
    <label>B: <input id="bval" type="number" min="0" max="255" value="87" /></label>
    <button id="run">Run</button>
    <span id="sum" style="margin-left:8px;font:14px monospace;"></span>
    <span id="flags" style="margin-left:8px;font:12px monospace;color:#444;"></span>
  </div>

  <div class="row" id="row"></div>
  <div class="legend">Legend: dark bubble = carry propagating; light bubble = blocked; SUB uses B⊕1 and Cin=1.</div>
  <div class="footer">Tip: try A=255, B=1 (ADD) and A=128, B=1 (SUB) to see overflow behavior.</div>

<script>
const N = 8;
const row = document.getElementById('row');
const modeEl = document.getElementById('mode');
const avalEl = document.getElementById('aval');
const bvalEl = document.getElementById('bval');
const runBtn = document.getElementById('run');
const sumEl = document.getElementById('sum');
const flagsEl = document.getElementById('flags');

function bit(v,i){ return (v>>i)&1; }

function stageEl(i){
  const s = document.createElement('div');
  s.className = 'stage add';
  s.innerHTML = `
    <div class="bitlabel">FA[${i}]</div>
    <div class="pins">
      <div class="io"><span>A${i}=</span><span class="a"></span></div>
      <div class="io"><span>B${i}=</span><span class="b"></span></div>
      <div class="io"><span>Ci=</span><span class="ci"></span></div>
      <div class="io"><span>S${i}=</span><span class="s"></span></div>
      <div class="io"><span>Co=</span><span class="co"></span></div>
    </div>
    <div class="carry"><div class="bubble"></div></div>
  `;
  return s;
}

for(let i=0;i<N;i++) row.appendChild(stageEl(i));

function run(){
  const A = Number(avalEl.value)|0;
  const B = Number(bvalEl.value)|0;
  const sub = modeEl.checked;
  const bIn = sub ? (~B)&0xFF : B;
  let cin = sub ? 1 : 0;
  let sum = 0;

  [...row.children].forEach((s,i)=>{
    s.className = 'stage ' + (sub?'sub':'add');
    s.querySelector('.a').textContent = bit(A,i);
    s.querySelector('.b').textContent = bit(bIn,i) + (sub?' (⊕1)':'');
    s.querySelector('.ci').textContent = cin;

    const ai = bit(A,i), bi = bit(bIn,i);
    const si = ai ^ bi ^ cin;
    const g = ai & bi;
    const p = ai ^ bi;
    const co = g | (p & cin);

    s.querySelector('.s').textContent = si;
    s.querySelector('.co').textContent = co;

    sum |= (si<<i);

    // animate carry bubble
    const c = s.querySelector('.carry');
    c.className = 'carry ' + (p? 'propagate':'blocked');
    const bub = c.querySelector('.bubble');
    bub.style.animation = 'none';
    void bub.offsetWidth; // reflow
    bub.style.animation = '';

    cin = co;
  });

  let cout = cin;
  // signed overflow detection (two's complement)
  const msbA = (A>>7)&1, msbB = ((sub?((~B)&0xFF):B)>>7)&1, msbS = (sum>>7)&1;
  const overflow = (msbA ^ msbS) & (msbB ^ msbS);

  sumEl.textContent = `Result = ${sum & 0xFF} (0x${(sum&0xFF).toString(16).padStart(2,'0')}) Cout=${cout}`;
  flagsEl.textContent = ` | OF=${overflow}  mode=${sub?'SUB':'ADD'}`;
}

runBtn.onclick = run;
run();
</script>
</body>
</html>
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>Timing Diagram — RCA Add/Sub</title>
<style>
  body{font-family:ui-monospace, SFMono-Regular, Menlo, monospace; margin:24px; color:#111}
  h1{font-size:20px;margin:0 0 8px}
  .row{margin:10px 0}
  svg{width:100%; max-width:960px; height:auto; border:1px solid #eee; border-radius:10px}
  .wave{fill:none; stroke-width:2}
  .a{stroke:#1f77b4}
  .b{stroke:#2ca02c}
  .mode{stroke:#ff7f0e}
  .sum{stroke:#111}
  .cout{stroke:#d62728}
  .of{stroke:#9467bd; stroke-dasharray:4 4}
  .dash{stroke-dasharray:800; stroke-dashoffset:800; animation: draw 1.4s ease forwards}
  @keyframes draw{to{stroke-dashoffset:0}}
  .legend{font-size:12px;color:#444}
</style>
</head>
<body>
<h1>Timing Diagram (animated)</h1>
<div class="legend">A,B inputs, MODE (0=ADD,1=SUB), SUM, COUT, and signed overflow (OF).</div>
<svg viewBox="0 0 960 260" role="img" aria-label="timing">
  <defs>
    <clipPath id="clip"><rect x="10" y="10" width="940" height="240"/></clipPath>
  </defs>
  <!-- grid -->
  <g stroke="#f0f0f0" stroke-width="1">
    <line x1="10" y1="50" x2="950" y2="50"/><line x1="10" y1="90" x2="950" y2="90"/>
    <line x1="10" y1="130" x2="950" y2="130"/><line x1="10" y1="170" x2="950" y2="170"/>
    <line x1="10" y1="210" x2="950" y2="210"/>
  </g>
  <!-- labels -->
  <g fill="#666" font-size="12">
    <text x="16" y="44">MODE</text>
    <text x="16" y="84">A</text>
    <text x="16" y="124">B</text>
    <text x="16" y="164">SUM</text>
    <text x="16" y="204">COUT / OF</text>
  </g>

  <!-- waves (simple square sequences) -->
  <g class="dash" clip-path="url(#clip)">
    <polyline class="wave mode" points="60,70 200,70 200,30 360,30 360,70 520,70 520,30 900,30 900,70"/>
    <polyline class="wave a"    points="60,110 180,110 180,70 300,70 300,110 420,110 420,70 900,70 900,110"/>
    <polyline class="wave b"    points="60,150 140,150 140,110 260,110 260,150 380,150 380,110 900,110 900,150"/>
    <!-- SUM responds after delay (ripple) -->
    <polyline class="wave sum"  points="60,190 240,190 240,150 420,150 420,190 600,190 600,150 900,150 900,190"/>
    <!-- COUT and OF -->
    <polyline class="wave cout" points="60,230 420,230 420,190 900,190 900,230"/>
    <polyline class="wave of"   points="60,230 240,230 240,190 420,190 420,230 600,230 600,190 900,190 900,230"/>
  </g>
</svg>
<p class="legend">The SUM/COUT edges lag inputs to illustrate carry-prop delay; OF toggles on signed overflow cases.</p>
</body>
</html>
