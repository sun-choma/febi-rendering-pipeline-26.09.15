<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue'

// Port of Chrome DevTools' official "jank" demo
// (googlechrome.github.io/devtools-samples/jank). Absolutely-positioned squares
// bounce via `top`. Motion is TIME-BASED: each square's position comes from a float
// integrated against elapsed wall-clock time, so a lower frame rate just drops frames
// (bigger jumps, same trajectory) — it never slows the motion down.
// Two versions of the SAME motion, differing only in the per-square DOM read:
//  · naive:     write top, then read offsetTop → the read forces a sync layout, per square
//  · optimized: write top, no layout read → the browser batches every write into one layout
const COUNT = 300
const version = ref<'naive' | 'optimized'>('naive')
const fps = ref(60)
const field = ref<HTMLElement>()

type Mover = { el: HTMLElement; pos: number; down: boolean }
let movers: Mover[] = []
const colors = ['#bd93f9', '#50fa7b', '#8be9fd', '#ffb86c', '#ff5555']
const SPEED = 110           // px / second
let FIELD_H = 260
const SQ = 8

function build() {
  const f = field.value!
  f.innerHTML = ''
  FIELD_H = f.clientHeight
  const frag = document.createDocumentFragment()
  movers = []
  for (let i = 0; i < COUNT; i++) {
    const el = document.createElement('div')
    el.className = 'sq'
    el.style.background = colors[i % colors.length]
    el.style.left = (Math.random() * 97).toFixed(2) + '%'   // free X, no column snapping
    const pos = Math.random() * ((FIELD_H - SQ) / 2)   // seed in the top half
    el.style.top = pos.toFixed(1) + 'px'
    frag.appendChild(el)
    movers.push({ el, pos, down: Math.random() > 0.5 })
  }
  f.appendChild(frag)
}

let raf = 0, frames = 0, lastTick = 0, lastFrame = 0
function frame(dt: number) {
  const max = FIELD_H - SQ
  const step = SPEED * dt
  const naive = version.value === 'naive'
  for (let i = 0; i < movers.length; i++) {
    const m = movers[i]
    m.pos += m.down ? step : -step
    if (m.pos <= 0) { m.pos = 0; m.down = true }
    else if (m.pos >= max) { m.pos = max; m.down = false }
    m.el.style.top = m.pos.toFixed(1) + 'px'   // write
    if (naive) {
      const y = m.el.offsetTop                 // read → forces sync layout each iteration
      if (i < 4) console.log('square top:', y) // "just logging the position" — the innocent-looking trap
    }
  }
}

function loop(now: number) {
  let dt = (now - lastFrame) / 1000
  lastFrame = now
  if (dt > 0.1) dt = 0.1        // clamp only a tab-return stall, not frame-rate differences
  frame(dt)
  frames++
  if (now - lastTick >= 400) { fps.value = Math.round(frames * 1000 / (now - lastTick)); frames = 0; lastTick = now }
  raf = requestAnimationFrame(loop)
}
function run(v: 'naive' | 'optimized') {
  version.value = v; frames = 0; lastTick = performance.now(); lastFrame = lastTick
  cancelAnimationFrame(raf); raf = requestAnimationFrame(loop)
}

// Start only once the field has a real height. If the deck loads on another slide,
// this component mounts hidden (clientHeight 0) — wait for it to become visible.
let started = false
let ro: ResizeObserver | null = null
function start() {
  if (started || !field.value || field.value.clientHeight < 20) return
  started = true
  ro?.disconnect(); ro = null
  build(); run('naive')
}

onMounted(async () => {
  await nextTick()
  start()
  if (!started && field.value) {
    ro = new ResizeObserver(() => start())
    ro.observe(field.value)
  }
})
onBeforeUnmount(() => { cancelAnimationFrame(raf); ro?.disconnect() })

function fpsClass(v: number) { return v >= 55 ? 'ok' : v >= 40 ? 'warn' : 'bad' }
</script>

<template>
  <div class="jank">
    <div class="topbar">
      <div class="seg">
        <button :class="{ active: version === 'naive' }" @click="run('naive')">Unoptimized</button>
        <button :class="{ active: version === 'optimized' }" @click="run('optimized')">Optimized</button>
      </div>
      <div class="spacer"></div>
      <div class="fpsbox" :class="fpsClass(fps)">
        <span class="n">{{ fps }}</span><span class="u">fps</span>
      </div>
    </div>
    <div ref="field" class="field"></div>
  </div>
</template>

<style scoped>
.jank { width: 100%; }
.topbar { display: flex; align-items: center; gap: 0.6rem; margin-bottom: 0.55rem; }
.seg { display: inline-flex; border: 1px solid #44475a; border-radius: 7px; overflow: hidden; }
.seg button { background: #282a36; color: #6272a4; border: none; padding: 6px 16px; font-weight: 600; cursor: pointer; font-size: 0.88rem; }
.seg button + button { border-left: 1px solid #44475a; }
.seg button.active { background: #44475a; color: #f8f8f2; }
.spacer { flex: 1; }
.fpsbox { font-family: 'Fira Code', monospace; display: flex; align-items: baseline; gap: 4px; color: #6272a4; }
.fpsbox .n { font-size: 1.7rem; font-weight: 700; }
.fpsbox .u { font-size: 0.8rem; opacity: 0.7; }
.fpsbox.ok { color: #50fa7b; }
.fpsbox.warn { color: #ffb86c; }
.fpsbox.bad { color: #ff5555; }
.field { position: relative; width: 100%; height: 220px; background: #21222c; border: 1px solid #44475a; border-radius: 8px; overflow: hidden; }
:deep(.sq) { position: absolute; width: 8px; height: 8px; border-radius: 2px; }
</style>
