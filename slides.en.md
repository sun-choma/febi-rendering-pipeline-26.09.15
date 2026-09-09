---
theme: dracula
title: Rendering Pipeline
transition: slide-left
---

# 🖥️ Rendering Pipeline

From HTML to pixels — what the browser does after the markup arrives.

---

# Same motion. Two ways.

<div class="hook-grid">

<div class="hook-left">
<JankSquares />
<div class="hook-q">Same squares, same motion — <b>two versions</b>. Why does <b class="bad">Unoptimized</b> crawl while <b class="good">Optimized</b> stays smooth?</div>
</div>

<div class="hook-spoiler">
<Spoiler>

Each frame loops over **all 300 squares** — and just logs each one's position. Harmless, right?

```js
// ❌ "just logging the position"
for (const s of squares) {
  s.style.top = y           // write
  console.log(s.offsetTop)  // read → layout NOW
}  // 300 reads → 300 layouts / frame

// ✅ optimized: drop the read → 1 layout / frame
```

Reading `offsetTop` **forces layout right now** — the innocent `console.log` triggers it once per square.

<span class="sp-punch">Not the log, not the write — **300 forced layouts a frame**.</span>

</Spoiler>
</div>

</div>

<style>
.hook-grid { display: grid; grid-template-columns: 1fr 300px; gap: 1.6rem; align-items: start; max-width: 900px; margin: 1.2rem auto 0; }
.hook-q { margin-top: 0.9rem; color: #f8f8f2; font-size: 0.92rem; text-align: center; }
.hook-q b { font-weight: 700; }
.hook-q .bad, .hook-spoiler .bad { color: #ff5555; }
.hook-q .good, .hook-spoiler .good { color: #50fa7b; }
.hook-spoiler { font-size: 0.82rem; line-height: 1.5; color: #6272a4; }
.hook-spoiler p { margin: 0.5rem 0; }
.hook-spoiler strong { color: #f8f8f2; }
.hook-spoiler .slidev-code { font-size: 0.58rem !important; line-height: 1.35 !important; margin: 0.45rem 0 !important; }
.hook-spoiler .slidev-code .line { min-height: 0 !important; }
.sp-punch { display: block; margin-top: 0.5rem; color: #ffb86c; }
.sp-punch strong { color: #ffb86c; }
</style>

---

# The pipeline, end to end

<div class="strip-wrap">
  <PipelineStrip :reveal="$clicks" :glow="[null, ['dom','cssom'], 'style', 'render', 'layout', 'paint', 'comp', 'screen'][$clicks] ?? null" />
</div>

<div class="one-liners">
<v-click>

- **DOM** — the structure
- **CSSOM** — the styles

</v-click>
<v-clicks>

- **Style** — match rules to elements, compute each element's final styles
- **Render Tree** — what's actually visible
- **Layout** — how big, and where
- **Paint** — fill in the pixels
- **Composite** — assemble the layers
- **🖥️** — the frame on screen

</v-clicks>
</div>

<!--
Orientation only. This is the map — don't rabbit-hole. Each click reveals one stage.
The payoff is the whole shape clicking into place; the deep dives come next.
-->

<style>
.strip-wrap { max-width: 900px; margin: 1.5rem auto 0.5rem; }
.one-liners { max-width: 900px; margin: 0 auto; padding-left: 21px; font-size: 0.92rem; }
.one-liners ul { list-style: none; padding: 0; margin: 0; }
.one-liners li { margin: 0.15rem 0; color: #f8f8f2; }
.one-liners strong { color: #bd93f9; }
</style>

---
layout: section
style: "--accent: #8be9fd"
---

<div class="section-num">03</div>

# Parsing → Trees

<div class="section-sub">HTML and CSS become the DOM and CSSOM — built in parallel.</div>

---

# Two sources → two trees

<div class="locator">
  <PipelineStrip :active="['dom','cssom']" accent="#8be9fd" />
</div>

<div class="parse">

<div class="cell src">

```html
<body>
  <h1>Hi</h1>
  <p>Text</p>
</body>
```

</div>
<div class="cell go">parse ▸</div>
<div class="cell">
<pre class="tree"><span class="t">html</span>
└─ <span class="t">body</span>
   ├─ <span class="t">h1</span> · <span class="s">"Hi"</span>
   └─ <span class="t">p</span> · <span class="s">"Text"</span></pre>
</div>

<div class="cell src">

```css
body { color: #eee }
h1   { font-size: 2rem }
```

</div>
<div class="cell go">parse ▸</div>
<div class="cell">
<pre class="tree"><span class="k">CSSOM</span>
├─ <span class="t">body</span> { <span class="p">color</span> }
└─ <span class="t">h1</span> { <span class="p">font-size</span> }</pre>
</div>

</div>

<div class="notes">

- ⏱ **Parallel** — both trees grow at once; neither waits for the other.
- 🌳 **The DOM ≠ your HTML** — invalid markup is repaired, implied tags (`html`, `body`, `tbody`) added. It's a live object model, not your source text.
- 🎨 **The CSSOM ≠ only your CSS** — your rules layer on top of the browser's default styles.

</div>

<style>
.locator { max-width: 500px; margin: 0.5rem auto 1.4rem; }
.parse {
  display: grid;
  grid-template-columns: auto auto 1fr;
  align-items: center;
  gap: 0.6rem 1.4rem;
  max-width: 760px;
  margin: 0 auto;
}
.parse .cell { min-width: 0; }
.parse .src :is(pre, .slidev-code) { margin: 0; }
.go { color: #8be9fd; font-family: 'Fira Code', monospace; font-size: 0.9rem; white-space: nowrap; }
.tree {
  margin: 0; font-family: 'Fira Code', monospace; font-size: 0.85rem; line-height: 1.5;
  color: #6272a4; background: #21222c; border: 1px solid #44475a; border-radius: 6px; padding: 0.6rem 0.9rem;
}
.tree .t { color: #ff79c6; }
.tree .s { color: #f1fa8c; }
.tree .k { color: #8be9fd; }
.tree .p { color: #50fa7b; }
.notes { max-width: 760px; margin: 1.3rem auto 0; font-size: 0.8rem; }
.notes ul { list-style: none; padding: 0; margin: 0; }
.notes li { color: #6272a4; margin: 0.3rem 0; line-height: 1.35; }
.notes strong { color: #f8f8f2; font-weight: 600; }
.notes code { color: #ff79c6; font-size: 0.9em; }
</style>

---

# CSS is render-blocking

<div class="locator">
  <PipelineStrip :active="['cssom']" accent="#8be9fd" />
</div>

<div class="rb-grid">

<div class="rb-text">

The **DOM** can build incrementally as HTML streams in.
The **CSSOM** can't — the cascade means the *last* rule can restyle the *first* element.

So the browser **holds the first paint** until all CSS is parsed. <span class="hl">CSS is render-blocking.</span>

The payoff: it paints **once, correctly** — instead of flashing unstyled content, then snapping into place.

</div>

<div class="rb-demo">

<div class="mock bad">
<div class="bar">✗ painting early</div>
<div class="page plain">
<div class="ph1">Sun*</div>
<div class="ptext">Frontend Biweekly</div>
<span class="plink">Learn more</span>
</div>
<div class="tag">flash of unstyled content</div>
</div>

<div class="mock good">
<div class="bar">✓ what the browser does</div>
<div class="page styled">
<div class="sh1">Sun*</div>
<div class="stext">Frontend Biweekly</div>
<span class="slink">Learn more →</span>
</div>
<div class="tag">waits, paints once</div>
</div>

</div>

</div>

<style>
.rb-grid { display: grid; grid-template-columns: 1fr auto; gap: 2.5rem; align-items: center; max-width: 860px; margin: 0.5rem auto 0; }
.rb-text { font-size: 0.95rem; color: #f8f8f2; line-height: 1.5; }
.rb-text p { margin: 0.7rem 0; }
.rb-text strong { color: #8be9fd; }
.rb-text .hl { color: #ffb86c; font-weight: 600; }
.rb-demo { display: flex; gap: 1.2rem; }
.mock { width: 150px; }
.mock .bar { font-size: 0.72rem; font-family: 'Fira Code', monospace; margin-bottom: 0.35rem; }
.mock.bad .bar { color: #ff5555; }
.mock.good .bar { color: #50fa7b; }
.mock .tag { font-size: 0.7rem; color: #6272a4; margin-top: 0.35rem; text-align: center; }
.page { border-radius: 6px; height: 120px; padding: 0.7rem 0.8rem; }
/* unstyled: browser defaults */
.page.plain { background: #ffffff; }
.page.plain .ph1 { font-family: 'Times New Roman', serif; color: #000; font-size: 1.4rem; font-weight: 700; }
.page.plain .ptext { font-family: 'Times New Roman', serif; color: #000; font-size: 0.85rem; margin: 0.3rem 0; }
.page.plain .plink { font-family: 'Times New Roman', serif; color: #00e; text-decoration: underline; font-size: 0.85rem; }
/* styled: your CSS applied */
.page.styled { background: #21222c; border: 1px solid #44475a; }
.page.styled .sh1 { color: #bd93f9; font-weight: 700; font-size: 1.4rem; }
.page.styled .stext { color: #6272a4; font-size: 0.85rem; margin: 0.3rem 0; }
.page.styled .slink { display: inline-block; margin-top: 0.4rem; color: #282a36; background: #50fa7b; padding: 3px 10px; border-radius: 5px; font-size: 0.78rem; font-weight: 600; }
</style>

---

# Style: the browser computes every value

<div class="locator">
  <PipelineStrip :active="['style']" accent="#8be9fd" />
</div>

<div class="st-grid">

<div class="cell">
<div class="st-lbl">One element</div>
<pre class="tree">&lt;p class="lead"&gt;</pre>
<div class="st-lbl">Rules that match</div>
<pre class="tree"><span class="t">p</span>     { <span class="p">color</span>: #eee }
<span class="t">.lead</span> { <span class="p">font-size</span>: 20px }
<span class="t">body</span>  { <span class="p">color</span>: #ccc } <span class="muted">← inherited</span></pre>
</div>

<div class="st-go">match<br>+ cascade ▸</div>

<div class="cell">
<div class="st-lbl">Computed style</div>
<pre class="tree"><span class="p">color</span>:     <span class="v">#eee</span>    <span class="muted">rule beats inherited</span>
<span class="p">font-size</span>: <span class="v">20px</span>   <span class="muted">.lead</span>
<span class="p">display</span>:   <span class="v">block</span>  <span class="muted">UA default</span>
<span class="p">margin</span>:    <span class="v">16px 0</span> <span class="muted">UA default</span>
<span class="muted">…and every other property</span></pre>
</div>

</div>

<div class="notes">

- Every element ends up with a value for **every** property — inherited or from the browser's defaults, even ones you never wrote.
- Change a class or a style and the browser **re-runs this** for the affected elements — that's **"Recalculate Style"** in DevTools.

</div>

<style>
.st-grid { display: grid; grid-template-columns: 1fr auto 1fr; align-items: center; gap: 1.4rem; max-width: 800px; margin: 0.5rem auto 0; }
.st-lbl { font-family: 'Fira Code', monospace; font-size: 0.75rem; color: #8be9fd; margin: 0.3rem 0; }
.st-go { text-align: center; color: #8be9fd; font-family: 'Fira Code', monospace; font-size: 0.85rem; line-height: 1.4; white-space: nowrap; }
.tree .v { color: #bd93f9; }
.tree .muted { color: #6272a4; font-size: 0.85em; }
</style>

---

# The render tree: only what's visible

<div class="locator">
  <PipelineStrip :active="['render']" accent="#8be9fd" />
</div>

<div class="rt-grid">

<div class="cell">
<div class="lbl">DOM</div>
<pre class="tree"><span class="t">body</span>
├─ <span class="t">nav</span>
├─ <span class="drop">div.banner</span>
└─ <span class="t">main</span></pre>
</div>

<div class="merge">
<div class="rule"><span class="sel">.banner</span> { <span class="p">display</span>: <span class="v">none</span> }</div>
<div class="drops">drops ▸</div>
</div>

<div class="cell">
<div class="lbl">Render tree</div>
<pre class="tree"><span class="t">body</span>
├─ <span class="t">nav</span>
├─ <span class="gone">div.banner ✕ dropped</span>
└─ <span class="t">main</span></pre>
</div>

</div>

<div class="rt-compare">

- <code class="c-drop">display: none</code> → **not in the render tree** · no box · no space · never painted
- <code class="c-keep">visibility: hidden</code> → **stays in the render tree** · keeps its box &amp; space · just not painted

</div>

<div class="rt-caveat">

⚠️ **But it's still in the DOM.** <code>display: none</code> is free to <em>render</em>, not free to <em>exist</em> — the node keeps its memory, JS references and listeners, and flipping it back on costs a fresh layout.

</div>

<style>
.rt-grid { display: grid; grid-template-columns: 1fr auto 1fr; align-items: center; gap: 1.5rem; max-width: 720px; margin: 0.5rem auto 0; }
.rt-grid .lbl { font-family: 'Fira Code', monospace; font-size: 0.8rem; color: #8be9fd; margin-bottom: 0.35rem; }
.rt-grid .tree .drop { color: #ffb86c; }
.rt-grid .tree .gone { color: #ff5555; text-decoration: line-through; opacity: 0.7; }
.merge { text-align: center; }
.merge .rule {
  display: inline-block; font-family: 'Fira Code', monospace; font-size: 0.72rem;
  background: #21222c; border: 1px solid #44475a; border-radius: 6px; padding: 0.4rem 0.6rem;
  white-space: nowrap; color: #f8f8f2;
}
.merge .rule .sel { color: #ffb86c; }
.merge .rule .p { color: #8be9fd; }
.merge .rule .v { color: #bd93f9; }
.merge .drops { color: #ff5555; font-family: 'Fira Code', monospace; font-size: 0.85rem; font-weight: 600; margin-top: 0.6rem; }
.rt-compare { max-width: 720px; margin: 1.6rem auto 0; font-size: 0.9rem; }
.rt-compare ul { list-style: none; padding: 0; margin: 0; }
.rt-compare li { color: #6272a4; margin: 0.45rem 0; }
.rt-compare strong { color: #f8f8f2; }
.rt-compare .c-drop { color: #ff5555; }
.rt-compare .c-keep { color: #50fa7b; }
.rt-caveat { max-width: 720px; margin: 0.9rem auto 0; font-size: 0.82rem; color: #6272a4; line-height: 1.4; }
.rt-caveat strong { color: #ffb86c; }
.rt-caveat code { color: #ff79c6; font-size: 0.9em; }
.rt-caveat em { color: #f8f8f2; font-style: normal; font-weight: 600; }
</style>

---
layout: section
style: "--accent: #ffb86c"
---

<div class="section-num">04</div>

# Layout & Reflow

<div class="section-sub">How big is every box, and where does it sit? And what does changing that cost?</div>

---

# Reflow: one change, many recalculations

<div class="locator">
  <PipelineStrip :active="['layout']" accent="#ffb86c" />
</div>

<div class="rf-grid">

<div class="rf-text">

**Layout** computes the geometry of every box — its **size** and its **position**.

**Reflow** = re-running layout after a geometric change.

It **cascades**: resize or move one box and its siblings, children, and ancestors may all have to be recomputed — sometimes the whole page.

<div class="rf-triggers">Triggers: <code>width</code>, <code>height</code>, <code>margin</code>, <code>font-size</code>, adding/removing nodes, window resize…</div>

</div>

<div class="rf-demo">

<div class="mini">
<div class="mtag">before</div>
<div class="doc">
<div class="row nav">nav</div>
<div class="row a">A</div>
<div class="row b">B</div>
<div class="row c">C</div>
</div>
</div>

<div class="rf-go">grow A ▸</div>

<div class="mini">
<div class="mtag">after</div>
<div class="doc">
<div class="row nav">nav</div>
<div class="row a tall reflowed">A</div>
<div class="row b reflowed">B</div>
<div class="row c reflowed">C</div>
</div>
</div>

</div>

</div>

<div class="rf-caption">Change A's height → B and C must move → <span class="hl">3 boxes recomputed, not 1</span>.</div>

<style>
.rf-grid { display: grid; grid-template-columns: 1fr auto; gap: 2.5rem; align-items: center; max-width: 860px; margin: 0.5rem auto 0; }
.rf-text { font-size: 0.95rem; color: #f8f8f2; line-height: 1.5; }
.rf-text p { margin: 0.6rem 0; }
.rf-text strong { color: #ffb86c; }
.rf-triggers { margin-top: 0.9rem; font-size: 0.78rem; color: #6272a4; }
.rf-triggers code { color: #ff79c6; }
.rf-demo { display: flex; align-items: center; gap: 1rem; }
.mini { width: 130px; }
.mini .mtag { font-family: 'Fira Code', monospace; font-size: 0.72rem; color: #6272a4; margin-bottom: 0.35rem; text-align: center; }
.doc { background: #21222c; border: 1px solid #44475a; border-radius: 6px; padding: 6px; display: flex; flex-direction: column; gap: 5px; }
.doc .row { border-radius: 4px; font-family: 'Fira Code', monospace; font-size: 0.72rem; color: #f8f8f2; display: flex; align-items: center; justify-content: center; }
.doc .nav { background: #44475a; height: 18px; }
.doc .a { background: #6272a4; height: 22px; }
.doc .a.tall { height: 46px; }
.doc .b { background: #6272a4; height: 22px; }
.doc .c { background: #6272a4; height: 22px; }
.doc .reflowed { outline: 2px solid #ffb86c; color: #ffb86c; background: #2b2a2a; }
.rf-go { color: #ffb86c; font-family: 'Fira Code', monospace; font-size: 0.8rem; white-space: nowrap; }
.rf-caption { text-align: center; color: #6272a4; font-size: 0.9rem; margin-top: 1.4rem; }
.rf-caption .hl { color: #ffb86c; font-weight: 600; }
</style>

---

# Layout thrashing — the read-after-write trap

<div class="lt-mech">A write just marks layout <b>dirty</b> (deferred). A read — <code>offsetHeight</code>, <code>getBoundingClientRect()</code> — forces the browser to <b>compute it right now</b>.</div>

<div class="lt-cols">

<div class="lt-col">
<div class="lt-tag bad">❌ interleaved</div>

```js
for (const el of items) {
  const h = el.offsetHeight  // read → layout!
  el.style.top = place(h)    // write → dirty
}
```

<div class="lt-count bad">⚡ one layout <b>per item</b> → N</div>
</div>

<div class="lt-col">
<div class="lt-tag good">✅ batched</div>

```js
const hs = items.map(el => el.offsetHeight)  // read all
items.forEach((el, i) =>
  el.style.top = place(hs[i]))               // write all
```

<div class="lt-count good">⚡ one layout <b>total</b> → 1</div>
</div>

</div>

<div class="lt-rule"><b>Batch reads, then writes.</b> Read while layout is clean, then dirty it once. <span class="dim">(libraries like <code>fastdom</code> automate this)</span></div>

<style>
.lt-mech { text-align: center; color: #6272a4; font-size: 0.88rem; max-width: 720px; margin: 0.5rem auto 1.4rem; line-height: 1.5; }
.lt-mech b { color: #f8f8f2; }
.lt-mech code { color: #ff79c6; }
.lt-cols { display: grid; grid-template-columns: 1fr 1fr; gap: 1.6rem; max-width: 760px; margin: 0 auto; }
.lt-tag { font-family: 'Fira Code', monospace; font-size: 0.8rem; margin-bottom: 0.35rem; }
.lt-tag.bad { color: #ff5555; }
.lt-tag.good { color: #50fa7b; }
.lt-cols .slidev-code { font-size: 0.76rem !important; margin: 0 !important; }
.lt-count { text-align: center; font-family: 'Fira Code', monospace; font-size: 0.82rem; margin-top: 0.6rem; }
.lt-count b { font-weight: 700; }
.lt-count.bad { color: #ff5555; }
.lt-count.good { color: #50fa7b; }
.lt-rule { text-align: center; color: #f8f8f2; font-size: 0.95rem; margin-top: 1.6rem; }
.lt-rule b { color: #ffb86c; }
.lt-rule .dim { color: #6272a4; font-size: 0.85em; }
.lt-rule code { color: #ff79c6; }
</style>

---
layout: section
style: "--accent: #50fa7b"
---

<div class="section-num">05</div>

# Paint & Repaint

<div class="section-sub">Now fill in the pixels — colors, text, borders, shadows. What does re-filling them cost?</div>

---

# Repaint: new pixels, same boxes

<div class="locator">
  <PipelineStrip :active="['paint']" accent="#50fa7b" />
</div>

<div class="rp-grid">

<div class="rp-text">

**Repaint** = re-filling an element's pixels when its *look* changes but its *box* doesn't.

<div class="path">
<span class="pst">Style</span><span class="arr">→</span><span class="pst skip">Layout</span><span class="arr">→</span><span class="pst on">Paint</span><span class="arr">→</span><span class="pst">Composite</span>
</div>
<div class="path-note">Geometry unchanged → <span class="hl">layout is skipped</span> → straight to paint.</div>

<div class="rp-triggers">Triggers: <code>color</code>, <code>background-color</code>, <code>visibility</code>, <code>box-shadow</code>, <code>outline</code></div>

</div>

<div class="rp-demo">

<div class="card dark">
<div class="ch">Sun*</div>
<div class="ct">Frontend Biweekly</div>
<div class="cb">Go</div>
</div>
<div class="rp-go">toggle theme ▸<br><small>repaint, no reflow</small></div>
<div class="card light">
<div class="ch">Sun*</div>
<div class="ct">Frontend Biweekly</div>
<div class="cb">Go</div>
</div>

</div>

</div>

<div class="rp-caveat">⚠️ Repaint isn't free either — a large area, or a <code>box-shadow</code> / blur / gradient, still costs. It just skips the layout stage.</div>

<style>
.rp-grid { display: grid; grid-template-columns: 1fr auto; gap: 2.5rem; align-items: center; max-width: 860px; margin: 0.5rem auto 0; }
.rp-text { font-size: 0.95rem; color: #f8f8f2; line-height: 1.5; }
.rp-text > strong { color: #50fa7b; }
.path { display: flex; align-items: center; gap: 0.5rem; margin: 1rem 0 0.4rem; font-family: 'Fira Code', monospace; font-size: 0.8rem; }
.pst { background: #21222c; border: 1px solid #44475a; border-radius: 5px; padding: 3px 9px; color: #6272a4; }
.pst.on { border-color: #50fa7b; color: #50fa7b; }
.pst.skip { text-decoration: line-through; opacity: 0.5; }
.arr { color: #6272a4; }
.path-note { font-size: 0.82rem; color: #6272a4; }
.path-note .hl { color: #50fa7b; }
.rp-triggers { margin-top: 0.9rem; font-size: 0.78rem; color: #6272a4; }
.rp-triggers code { color: #ff79c6; }
.rp-demo { display: flex; align-items: center; gap: 0.9rem; }
.card { width: 110px; border-radius: 8px; padding: 0.7rem 0.8rem; }
.card.dark { background: #21222c; border: 1px solid #44475a; }
.card.dark .ch { color: #bd93f9; }
.card.dark .ct { color: #6272a4; }
.card.dark .cb { background: #50fa7b; color: #282a36; }
.card.light { background: #f8f8f2; border: 1px solid #dcdce4; }
.card.light .ch { color: #6b3fd4; }
.card.light .ct { color: #6b7280; }
.card.light .cb { background: #2e9e57; color: #f8f8f2; }
.card .ch { font-weight: 700; font-size: 1.1rem; }
.card .ct { font-size: 0.78rem; margin: 0.25rem 0 0.5rem; }
.card .cb { display: inline-block; padding: 3px 12px; border-radius: 5px; font-size: 0.78rem; font-weight: 600; }
.rp-go { color: #50fa7b; font-family: 'Fira Code', monospace; font-size: 0.78rem; text-align: center; white-space: nowrap; }
.rp-go small { color: #6272a4; font-size: 0.85em; }
.rp-caveat { max-width: 860px; margin: 1.5rem auto 0; font-size: 0.82rem; color: #6272a4; text-align: center; }
.rp-caveat code { color: #ff79c6; }
</style>

---
layout: section
style: "--accent: #bd93f9"
---

<div class="section-num">06</div>

# Composite

<div class="section-sub">The final assembly — stack the layers on the GPU. And why <code>transform</code> & <code>opacity</code> are nearly free.</div>

---

# Composite: stacking pre-painted layers

<div class="locator">
  <PipelineStrip :active="['comp']" accent="#bd93f9" />
</div>

<div class="cp-grid">

<div class="cp-scene">
<CompositeLayers />
<div class="scene-cap">3 pre-painted layers → <span class="gpu">GPU stacks them</span> → 🖥️ one frame</div>
<div class="scene-hint">⟳ drag to rotate · double-click to reset</div>
</div>

<div class="cp-text">

<code>transform</code> repositions a whole layer · <code>opacity</code> fades one — the layer's pixels are **already painted**.

<div class="path">
<span class="pst skip">Layout</span><span class="arr">→</span><span class="pst skip">Paint</span><span class="arr">→</span><span class="pst on">Composite</span>
</div>
<div class="path-note">Skip layout <b>and</b> paint — only the compositor runs.</div>

<div class="cp-gpu">🧠 <b>"GPU-accelerated" isn't magic</b> — the compositor just reassembles layers it already has, on the GPU, <b>off the main thread</b>. It's a cheaper stage, not a faster version of the same work.</div>

</div>

</div>

<style>
.cp-grid { display: grid; grid-template-columns: 220px 1fr; gap: 2rem; align-items: start; max-width: 820px; margin: 3rem auto 0; }
.cp-text { padding-top: 0; }
.cp-text > :first-child { margin-top: 0; }
.cp-scene { text-align: center; }
.scene-cap { font-size: 0.75rem; color: #6272a4; margin-top: 0.8rem; }
.scene-cap .gpu { color: #bd93f9; }
.scene-hint { font-size: 0.68rem; color: #6272a4; opacity: 0.7; margin-top: 0.3rem; }
.cp-text { font-size: 0.95rem; color: #f8f8f2; line-height: 1.5; }
.cp-text > code { color: #ff79c6; }
.cp-text strong { color: #bd93f9; }
.path { display: flex; align-items: center; gap: 0.5rem; margin: 1rem 0 0.35rem; font-family: 'Fira Code', monospace; font-size: 0.8rem; }
.path .pst { background: #21222c; border: 1px solid #44475a; border-radius: 5px; padding: 3px 9px; color: #6272a4; }
.path .pst.on { border-color: #bd93f9; color: #bd93f9; }
.path .pst.skip { text-decoration: line-through; opacity: 0.5; }
.path .arr { color: #6272a4; }
.path-note { font-size: 0.82rem; color: #6272a4; }
.path-note b { color: #bd93f9; }
.cp-gpu { margin-top: 1rem; font-size: 0.8rem; color: #6272a4; line-height: 1.45; border-left: 3px solid #bd93f9; padding-left: 0.8rem; }
.cp-gpu b { color: #f8f8f2; }
</style>

---

# Same motion, two costs

<div class="tw">

<div class="twcol" style="--c: #ff5555">
<div class="twlabel">move with <code>left</code></div>
<div class="track"><div class="tbox move-left"></div></div>
<div class="tbadges"><span class="badge on">Layout</span><span class="badge on">Paint</span><span class="badge on">Composite</span></div>
<div class="twcost">every frame → layout + paint + composite</div>
</div>

<div class="twcol" style="--c: #50fa7b">
<div class="twlabel">move with <code>transform</code></div>
<div class="track"><div class="tbox move-tf"></div></div>
<div class="tbadges"><span class="badge off">Layout</span><span class="badge off">Paint</span><span class="badge on">Composite</span></div>
<div class="twcost">every frame → just composite</div>
</div>

</div>

<div class="tcap">Pixel-identical motion — but <code>left</code> re-runs layout &amp; paint every frame, while <code>transform</code> just repositions a finished layer. <span class="hl">That's the compositor payoff.</span></div>

<style>
.tw { display: grid; grid-template-columns: repeat(2, 1fr); gap: 3rem; max-width: 760px; margin: 1.8rem auto 0; }
.twcol { text-align: center; }
.twlabel { font-size: 0.85rem; color: #f8f8f2; margin-bottom: 0.7rem; }
.twlabel code { color: var(--c); font-family: 'Fira Code', monospace; }
.track { position: relative; width: 300px; height: 60px; margin: 0 auto; background: #21222c; border: 1px solid #44475a; border-radius: 8px; }
.tbox { position: absolute; top: 8px; left: 8px; width: 44px; height: 44px; border-radius: 8px; background: #bd93f9; }
.move-left { animation: mvl 2s ease-in-out infinite; }
.move-tf { animation: mvt 2s ease-in-out infinite; }
@keyframes mvl { 0%, 100% { left: 8px } 50% { left: 248px } }
@keyframes mvt { 0%, 100% { transform: translateX(0) } 50% { transform: translateX(240px) } }
.tbadges { display: flex; justify-content: center; gap: 5px; margin-top: 0.9rem; }
.badge { font-size: 0.64rem; font-family: 'Fira Code', monospace; padding: 3px 7px; border-radius: 4px; border: 1px solid #44475a; }
.badge.on { background: color-mix(in srgb, var(--c) 22%, transparent); border-color: var(--c); color: var(--c); }
.badge.off { color: #6272a4; text-decoration: line-through; opacity: 0.6; }
.twcost { margin-top: 0.8rem; font-size: 0.78rem; color: var(--c); font-family: 'Fira Code', monospace; }
.tcap { text-align: center; color: #6272a4; font-size: 0.9rem; margin-top: 2rem; max-width: 680px; margin-left: auto; margin-right: auto; }
.tcap code { color: #ff79c6; }
.tcap .hl { color: #50fa7b; font-weight: 600; }
</style>

---
layout: section
style: "--accent: #ff5555"
---

<div class="section-num">07</div>

# Key takeaways

<div class="section-sub">See it yourself, the mental model, and the cost cheat sheet.</div>

---

# Catch each stage red-handed

<div class="locator">
  <PipelineStrip :active="['layout','paint','comp']" accent="#ff5555" />
</div>

<div class="dt-intro">Now that you know what each stage does — here's how to <b>see</b> each one:</div>

<div class="dt-grid">

<div class="dt-card" style="--c: #50fa7b">
<div class="dt-mock"><div class="m-ui"><span class="m-flash"></span></div></div>
<div class="dt-name">Paint flashing</div>
<div class="dt-where">Rendering tab</div>
<div class="dt-what">Green flashes mark regions that just <b>repainted</b>.</div>
<div class="dt-catch">→ Paint</div>
</div>

<div class="dt-card" style="--c: #ffb86c">
<div class="dt-mock"><div class="m-ui m-layer"></div></div>
<div class="dt-name">Layer borders</div>
<div class="dt-where">Rendering tab</div>
<div class="dt-what">Orange outlines show each <b>compositor layer</b>.</div>
<div class="dt-catch">→ Composite</div>
</div>

<div class="dt-card" style="--c: #bd93f9">
<div class="dt-mock"><div class="m-track"><span class="m-seg lay">Layout</span><span class="m-seg pnt">Paint</span></div></div>
<div class="dt-name">Performance recording</div>
<div class="dt-where">Performance tab</div>
<div class="dt-what">See <b>Layout</b> vs <b>Paint</b> blocks in a frame.</div>
<div class="dt-catch">→ Layout</div>
</div>

</div>

<div class="dt-note">💡 Toggle all three on our opener's demo — the <b>jank from earlier</b>, now visible on screen.</div>

<style>
.locator { max-width: 460px; margin: 0.2rem auto 0.8rem; }
.dt-intro { text-align: center; color: #6272a4; font-size: 0.85rem; margin: 0.2rem auto 0.8rem; }
.dt-intro b { color: #ff5555; }
.dt-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.2rem; max-width: 800px; margin: 0 auto; }
.dt-card { background: #21222c; border: 1px solid #44475a; border-top: 3px solid var(--c); border-radius: 8px; padding: 0.7rem; text-align: center; }
.dt-mock { height: 46px; display: flex; align-items: center; justify-content: center; margin-bottom: 0.5rem; }
.m-ui { position: relative; width: 84px; height: 48px; background: #282a36; border: 1px solid #44475a; border-radius: 5px; }
.m-flash { position: absolute; right: 8px; bottom: 8px; width: 34px; height: 18px; background: rgba(80,250,123,0.55); border: 1px solid #50fa7b; border-radius: 3px; }
.m-layer { outline: 2px dashed #ffb86c; outline-offset: 2px; }
.m-track { display: flex; gap: 4px; width: 130px; }
.m-seg { font-family: 'Fira Code', monospace; font-size: 0.6rem; color: #282a36; padding: 4px 0; border-radius: 3px; text-align: center; }
.m-seg.lay { background: #bd93f9; flex: 2; }
.m-seg.pnt { background: #50fa7b; flex: 1; }
.dt-name { color: #f8f8f2; font-weight: 600; font-size: 0.92rem; }
.dt-where { color: var(--c); font-family: 'Fira Code', monospace; font-size: 0.7rem; margin: 0.15rem 0 0.5rem; }
.dt-what { color: #6272a4; font-size: 0.78rem; line-height: 1.4; }
.dt-what b { color: #f8f8f2; }
.dt-catch { margin-top: 0.6rem; color: var(--c); font-family: 'Fira Code', monospace; font-size: 0.8rem; font-weight: 700; }
.dt-note { text-align: center; color: #6272a4; font-size: 0.82rem; margin-top: 1rem; }
.dt-note b { color: #ff5555; }
</style>

---

# Walk away with this

<div class="kt">

- 🧭 **It's a pipeline, in order** — HTML/CSS → trees → Style → Layout → Paint → Composite → pixels. Knowing the order is the whole game.
- 💸 **Cost = how many stages re-run** — a change re-runs everything from its stage onward. The earlier the stage, the more expensive.
- 🎯 **Animate on the cheap path** — <code>transform</code> & <code>opacity</code> only composite. Reach for them for motion; avoid animating layout properties.
- 🧹 **Don't make the browser redo work** — batch DOM reads, then writes, so layout runs once instead of per element.

</div>

<div class="kt-foot">None of this means micro-optimize everything — it means <b>know what's expensive</b>, so you recognize what scales badly.</div>

<style>
.kt { max-width: 760px; margin: 1.6rem auto 0; }
.kt ul { list-style: none; padding: 0; margin: 0; }
.kt li { color: #f8f8f2; font-size: 0.98rem; line-height: 1.45; margin: 0.9rem 0; }
.kt strong { color: #bd93f9; }
.kt code { color: #ff79c6; }
.kt-foot { max-width: 760px; margin: 1.6rem auto 0; text-align: center; color: #6272a4; font-size: 0.85rem; }
.kt-foot b { color: #f8f8f2; }
</style>

---

# So — what does each change cost?

<div class="fin-strip">
  <PipelineStrip />
</div>

<div class="cs-wrap">

<table class="cs">
<thead>
<tr><th>Change this…</th><th>Layout</th><th>Paint</th><th>Composite</th></tr>
</thead>
<tbody>
<tr class="r-hi"><td><code>width</code>, <code>top</code>, <code>font-size</code></td><td class="y">✓</td><td class="y">✓</td><td class="y">✓</td></tr>
<tr class="r-mid"><td><code>background-color</code>, <code>color</code></td><td class="n">·</td><td class="y">✓</td><td class="y">✓</td></tr>
<tr class="r-lo"><td><code>transform</code>, <code>opacity</code></td><td class="n">·</td><td class="n">·</td><td class="y">✓</td></tr>
</tbody>
</table>

<div class="cs-legend">A ✓ means that stage must <b>re-run</b> — the more ✓, the more expensive. Our smooth demo used <code>transform</code>: the bottom row — <b>Composite only</b>.</div>

<div class="cs-foot">Re-running <b>Layout</b> is called <i>reflow</i>; re-running <b>Paint</b> is <i>repaint</i>. · Full reference: <a class="url" href="https://csstriggers.com" target="_blank">csstriggers.com</a></div>

</div>

<style>
.fin-strip { max-width: 620px; margin: 1.2rem auto 0.2rem; }
.cs-legend code { color: #ff79c6; }
.cs-wrap { max-width: 720px; margin: 1rem auto 0; }
.cs { width: 100%; border-collapse: collapse; font-size: 0.95rem; }
.cs th, .cs td { padding: 0.7rem 1rem; text-align: center; border-bottom: 1px solid #44475a; }
.cs th:first-child, .cs td:first-child { text-align: left; }
.cs thead th { color: #f8f8f2; font-family: 'Fira Code', monospace; font-size: 0.85rem; border-bottom: 2px solid #6272a4; }
.cs td code { color: #ff79c6; font-size: 0.85em; }
.cs .y { color: #ff5555; font-weight: 700; }
.cs .n { color: #44475a; }
.cs .r-hi { background: rgba(255,85,85,0.07); }
.cs .r-lo { background: rgba(80,250,123,0.07); }
.cs .r-lo .y { color: #50fa7b; }
.cs .r-mid .y:last-of-type, .cs .r-mid .y { color: #ffb86c; }
.cs-legend { text-align: center; color: #6272a4; font-size: 0.85rem; margin-top: 1.1rem; }
.cs-legend b { color: #f8f8f2; }
.cs-foot { text-align: center; color: #6272a4; font-size: 0.75rem; margin-top: 0.8rem; }
.cs-foot b { color: #f8f8f2; }
.cs-foot i { color: #8be9fd; font-style: normal; }
.cs-foot .url { color: #8be9fd; text-decoration: none; border-bottom: 1px solid rgba(139,233,253,0.4); }
.cs-foot .url:hover { border-bottom-color: #8be9fd; }
</style>

---
layout: center
class: text-center
---

# Hardware isn't the problem 🖥️

<div class="cta">Understand the pipeline, build for the weakest device — performance is accessibility.</div>

<div class="pills">

<a class="pill" href="https://web.dev/articles/rendering-performance" target="_blank">
<span class="i-ph-link"/>web.dev — Rendering performance</a>

<a class="pill" href="https://csstriggers.com" target="_blank">
<span class="i-ph-link"/>csstriggers.com</a>

<a class="pill" href="https://developer.chrome.com/blog/inside-browser-part1" target="_blank">
<span class="i-ph-link"/>Inside look at modern web browsers</a>

<a class="pill" href="https://github.com/wilsonpage/fastdom" target="_blank">
<span class="i-ph-link"/>fastdom</a>

</div>

<style>
.cta { font-size: 1.05rem; color: #6272a4; margin-top: 0.75rem; }
.pills {
  display: flex; flex-wrap: wrap; gap: 0.6rem;
  justify-content: center; margin-top: 3rem;
}
.pill {
  display: flex; align-items: center; gap: 0.4rem;
  background: #282a36; border: 1px solid #44475a; border-radius: 999px;
  padding: 0.4rem 0.9rem;
  font-size: 0.78rem; color: #f8f8f2; text-decoration: none;
  transition: border-color 0.15s, color 0.15s;
}
.pill:hover { border-color: #bd93f9; color: #bd93f9; }
</style>
