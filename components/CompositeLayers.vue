<script setup lang="ts">
import { ref } from 'vue'

// Freeform drag-to-rotate for the compositing layer stack.
// Horizontal drag spins (rotateZ), vertical drag tilts (rotateX). Double-click resets.
const REST_X = 50, REST_Z = -24
const rotX = ref(REST_X)
const rotZ = ref(REST_Z)
const dragging = ref(false)
let lastX = 0, lastY = 0

function down(e: PointerEvent) {
  dragging.value = true
  lastX = e.clientX; lastY = e.clientY
  ;(e.currentTarget as HTMLElement).setPointerCapture(e.pointerId)
}
function move(e: PointerEvent) {
  if (!dragging.value) return
  rotZ.value -= (e.clientX - lastX) * 0.45
  rotX.value = Math.min(88, Math.max(2, rotX.value - (e.clientY - lastY) * 0.45))
  lastX = e.clientX; lastY = e.clientY
}
function up(e: PointerEvent) {
  dragging.value = false
  ;(e.currentTarget as HTMLElement).releasePointerCapture?.(e.pointerId)
}
function reset() { rotX.value = REST_X; rotZ.value = REST_Z }
</script>

<template>
  <div
    class="scene" :class="{ dragging }"
    @pointerdown="down" @pointermove="move" @pointerup="up" @pointercancel="up" @dblclick="reset"
  >
    <div class="stack" :style="{ transform: `translateY(20px) rotateX(${rotX}deg) rotateZ(${rotZ}deg)` }">
      <div class="layer l1">background</div>
      <div class="layer l2">page content</div>
      <div class="layer l3">moving element</div>
    </div>
  </div>
</template>

<style scoped>
.scene {
  perspective: 780px; width: 230px; height: 185px; margin: 0 auto;
  display: flex; align-items: center; justify-content: center;
  cursor: grab; touch-action: none; user-select: none;
}
.scene.dragging { cursor: grabbing; }
.stack {
  position: relative; width: 148px; height: 80px;
  transform-style: preserve-3d;
  transform: translateY(20px) rotateX(50deg) rotateZ(-24deg);
  transition: transform .5s ease;
}
.scene.dragging .stack { transition: none; }
.layer {
  position: absolute; inset: 0; border-radius: 7px;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Fira Code', monospace; font-size: 0.66rem; color: #f8f8f2;
  backface-visibility: hidden;
}
.l1 { transform: translateZ(0);    background: rgba(68,71,90,0.6);   border: 1.5px solid #44475a; }
.l2 { transform: translateZ(38px); background: rgba(98,114,164,0.5);  border: 1.5px solid #6272a4; }
.l3 { transform: translateZ(76px); background: rgba(189,147,249,0.42); border: 2px solid #bd93f9; color: #bd93f9; font-weight: 700; }
</style>
