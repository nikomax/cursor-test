<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

const cursor = ref<HTMLDivElement | null>(null)
const ring = ref<HTMLDivElement | null>(null)
const visible = ref(false)

let raf = 0
let x = 0
let y = 0
let tx = 0
let ty = 0
let scale = 1
let targetScale = 1
let onMove: ((event: PointerEvent) => void) | null = null
let onLeave: (() => void) | null = null

onMounted(() => {
  if (!window.matchMedia('(pointer: fine)').matches) return
  document.documentElement.classList.add('has-magnetic-cursor')

  onMove = (event: PointerEvent) => {
    visible.value = true
    tx = event.clientX
    ty = event.clientY
    const target = event.target as HTMLElement | null
    const interactive = target?.closest('a, button, .dot, .nav-btn')
    targetScale = interactive ? 2.2 : 1
  }

  onLeave = () => {
    visible.value = false
  }

  const tick = () => {
    raf = requestAnimationFrame(tick)
    x += (tx - x) * 0.18
    y += (ty - y) * 0.18
    scale += (targetScale - scale) * 0.12

    if (cursor.value) {
      cursor.value.style.transform = `translate3d(${tx}px, ${ty}px, 0) scale(${scale})`
    }
    if (ring.value) {
      ring.value.style.transform = `translate3d(${x}px, ${y}px, 0) scale(${scale * 0.85})`
    }
  }

  window.addEventListener('pointermove', onMove)
  document.documentElement.addEventListener('pointerleave', onLeave)
  tick()
})

onUnmounted(() => {
  cancelAnimationFrame(raf)
  if (onMove) window.removeEventListener('pointermove', onMove)
  if (onLeave) document.documentElement.removeEventListener('pointerleave', onLeave)
  document.documentElement.classList.remove('has-magnetic-cursor')
})
</script>

<template>
  <div class="cursor-root" aria-hidden="true">
    <div ref="cursor" class="cursor-dot" :class="{ visible }" />
    <div ref="ring" class="cursor-ring" :class="{ visible }" />
  </div>
</template>

<style scoped>
.cursor-root {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 9999;
}

.cursor-dot,
.cursor-ring {
  position: fixed;
  top: 0;
  left: 0;
  border-radius: 50%;
  opacity: 0;
  transition: opacity 0.25s ease;
  will-change: transform;
}

.cursor-dot.visible,
.cursor-ring.visible {
  opacity: 1;
}

.cursor-dot {
  width: 8px;
  height: 8px;
  margin: -4px 0 0 -4px;
  background: #d6ff4b;
  mix-blend-mode: difference;
}

.cursor-ring {
  width: 36px;
  height: 36px;
  margin: -18px 0 0 -18px;
  border: 1px solid rgba(214, 255, 75, 0.65);
}

@media (pointer: coarse) {
  .cursor-root {
    display: none;
  }
}
</style>
