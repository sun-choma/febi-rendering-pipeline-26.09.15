<script setup lang="ts">
import { ref } from 'vue'
defineProps<{ label?: string }>()
const shown = ref(false)
</script>

<template>
  <div class="spoiler">
    <button class="sp-btn" :class="{ open: shown }" @click="shown = !shown">
      <span class="ic">{{ shown ? '🔓' : '🔒' }}</span>
      {{ label || 'The catch' }}
      <span class="hint">{{ shown ? '' : '— click to reveal' }}</span>
    </button>
    <div class="sp-content" v-show="shown"><slot /></div>
  </div>
</template>

<style scoped>
.spoiler { background: #21222c; border: 1px solid #44475a; border-radius: 8px; overflow: hidden; }
.sp-btn {
  width: 100%; text-align: left; background: #282a36; color: #f8f8f2; border: none;
  padding: 0.65rem 0.9rem; font-size: 0.9rem; font-weight: 600; cursor: pointer;
  display: flex; align-items: center; gap: 0.5rem;
}
.sp-btn.open { border-bottom: 1px solid #44475a; }
.sp-btn .hint { color: #6272a4; font-weight: 400; font-size: 0.82rem; }
.sp-content { padding: 0.85rem 1rem; }
</style>
