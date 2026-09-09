<script setup lang="ts">
import { computed } from 'vue'

// The recurring rendering-pipeline diagram.
//  · full size  -> §2 star (reveal stage-by-stage with `reveal`)
//  · shrunk      -> §3–7 header "you are here" (highlight one stage with `active`)
const props = withDefaults(defineProps<{
  reveal?: number            // steps 0..7 visible; default = all
  active?: string | string[] | null  // header "you are here": highlight these, dim the rest
  glow?: string | string[] | null    // reveal highlight: accent this, others stay bright
  accent?: string            // highlight colour (per-section)
}>(), { reveal: 99, active: null, glow: null, accent: '#bd93f9' })

interface Node { id: string; lines: string[]; cx: number; cy: number; step: number; kind: 'src' | 'tree' | 'stage' | 'screen' }

// step 0 = HTML/CSS sources · step 1 = both trees together · then one step per stage
const nodes: Node[] = [
  { id: 'html',   lines: ['HTML'],            cx: 60,   cy: 55,  step: 0, kind: 'src' },
  { id: 'css',    lines: ['CSS'],             cx: 60,   cy: 155, step: 0, kind: 'src' },
  { id: 'dom',    lines: ['DOM'],             cx: 180,  cy: 55,  step: 1, kind: 'tree' },
  { id: 'cssom',  lines: ['CSSOM'],           cx: 180,  cy: 155, step: 1, kind: 'tree' },
  { id: 'style',  lines: ['Style'],           cx: 330,  cy: 105, step: 2, kind: 'stage' },
  { id: 'render', lines: ['Render', 'Tree'],  cx: 500,  cy: 105, step: 3, kind: 'stage' },
  { id: 'layout', lines: ['Layout'],          cx: 650,  cy: 105, step: 4, kind: 'stage' },
  { id: 'paint',  lines: ['Paint'],           cx: 785,  cy: 105, step: 5, kind: 'stage' },
  { id: 'comp',   lines: ['Composite'],       cx: 925,  cy: 105, step: 6, kind: 'stage' },
  { id: 'screen', lines: ['🖥️'],              cx: 1055, cy: 105, step: 7, kind: 'screen' },
]
const byId = Object.fromEntries(nodes.map(n => [n.id, n]))

const edges = [
  { from: 'html',   to: 'dom',    step: 1 },
  { from: 'css',    to: 'cssom',  step: 1 },
  { from: 'dom',    to: 'style',  step: 2 },
  { from: 'cssom',  to: 'style',  step: 2 },
  { from: 'style',  to: 'render', step: 3 },
  { from: 'render', to: 'layout', step: 4 },
  { from: 'layout', to: 'paint',  step: 5 },
  { from: 'paint',  to: 'comp',   step: 6 },
  { from: 'comp',   to: 'screen', step: 7 },
]

function halfW(n: Node) { return n.kind === 'src' ? 34 : n.kind === 'tree' ? 42 : n.kind === 'stage' ? 52 : 22 }
function halfH(n: Node) { return n.kind === 'stage' ? 24 : n.kind === 'screen' ? 24 : 20 }

function toSet(v: string | string[] | null) { return new Set(v == null ? [] : Array.isArray(v) ? v : [v]) }
const activeSet = computed(() => toSet(props.active))
const glowSet = computed(() => toSet(props.glow))

// 'on'  = highlighted (accent)   'off' = dimmed   'bright' = normal
function state(id: string): 'bright' | 'on' | 'off' {
  if (props.active != null) return activeSet.value.has(id) ? 'on' : 'off'  // header: dim the rest
  if (glowSet.value.has(id)) return 'on'                                    // reveal: glow, no dimming
  return 'bright'
}
function shown(step: number) { return step <= props.reveal }

function nodeFill(id: string) { return state(id) === 'off' ? '#21222c' : '#343746' }
function nodeStroke(id: string) { return state(id) === 'on' ? props.accent : state(id) === 'off' ? '#313442' : '#6272a4' }
function nodeStrokeW(id: string) { return state(id) === 'on' ? 3 : 1.5 }
function textFill(id: string) { const s = state(id); return s === 'on' ? props.accent : s === 'off' ? '#6272a4' : '#f8f8f2' }
function rectStyle(id: string) {
  return state(id) === 'on' ? { filter: `drop-shadow(0 0 6px ${props.accent})`, transition: 'filter .3s' } : { transition: 'filter .3s' }
}

function edge(e: { from: string; to: string }) {
  const a = byId[e.from], b = byId[e.to]
  return { x1: a.cx + halfW(a) + 4, y1: a.cy, x2: b.cx - halfW(b) - 6, y2: b.cy }
}
</script>

<template>
  <svg class="pipe" viewBox="0 0 1130 210" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <marker id="pipe-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" fill="#6272a4" />
      </marker>
    </defs>

    <g>
      <line v-for="e in edges" :key="e.from + e.to"
        :x1="edge(e).x1" :y1="edge(e).y1" :x2="edge(e).x2" :y2="edge(e).y2"
        stroke="#6272a4" stroke-width="2" marker-end="url(#pipe-arrow)"
        :style="{ opacity: shown(e.step) ? 1 : 0, transition: 'opacity .3s' }" />
    </g>

    <g v-for="n in nodes" :key="n.id" :style="{ opacity: shown(n.step) ? 1 : 0, transition: 'opacity .3s' }">
      <template v-if="n.kind === 'screen'">
        <text :x="n.cx" :y="n.cy" :style="{ fontSize: '38px' }" text-anchor="middle" dominant-baseline="central">{{ n.lines[0] }}</text>
      </template>
      <template v-else>
        <rect :x="n.cx - halfW(n)" :y="n.cy - halfH(n)" :width="halfW(n) * 2" :height="halfH(n) * 2"
          rx="8" :fill="nodeFill(n.id)" :stroke="nodeStroke(n.id)" :stroke-width="nodeStrokeW(n.id)" :style="rectStyle(n.id)" />
        <text :x="n.cx" :y="n.cy"
          :style="{ fontSize: (n.kind === 'src' || n.kind === 'tree' ? 16 : 17) + 'px', fontFamily: `'Fira Code', monospace`, fontWeight: 600 }"
          text-anchor="middle" dominant-baseline="central" :fill="textFill(n.id)">
          <template v-if="n.lines.length === 1">{{ n.lines[0] }}</template>
          <template v-else>
            <tspan :x="n.cx" dy="-0.55em">{{ n.lines[0] }}</tspan>
            <tspan :x="n.cx" dy="1.1em">{{ n.lines[1] }}</tspan>
          </template>
        </text>
      </template>
    </g>
  </svg>
</template>

<style scoped>
.pipe { width: 100%; height: auto; display: block; }
</style>
