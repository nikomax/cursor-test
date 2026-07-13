<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'

interface Slide {
  image: string
  title: string
  caption: string
}

const slides: Slide[] = [
  {
    image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=1400&q=80',
    title: 'Alpine Peaks',
    caption: 'Snow-capped mountains under a clear sky',
  },
  {
    image: 'https://images.unsplash.com/photo-1469474968028-56623f02e42e?w=1400&q=80',
    title: 'Golden Valley',
    caption: 'Sunlight streaming through rolling hills',
  },
  {
    image: 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=1400&q=80',
    title: 'Forest Path',
    caption: 'A quiet trail through ancient woodland',
  },
  {
    image: 'https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?w=1400&q=80',
    title: 'Misty Dawn',
    caption: 'Fog settling over a tranquil lake',
  },
  {
    image: 'https://images.unsplash.com/photo-1518837695005-2083093ee35b?w=1400&q=80',
    title: 'Ocean Waves',
    caption: 'Turquoise waters meeting the shore',
  },
]

const COLS = 14
const ROWS = 8
const BREAK_DURATION = 0.55
const JOIN_DURATION = 0.65

type TransitionPhase = 'idle' | 'breaking' | 'joining'

interface Piece {
  mesh: THREE.Mesh
  homePosition: THREE.Vector3
  homeRotation: THREE.Euler
  scatterPosition: THREE.Vector3
  scatterRotation: THREE.Euler
}

const containerRef = ref<HTMLDivElement | null>(null)
const currentIndex = ref(0)
const isTransitioning = ref(false)

let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let pieces: Piece[] = []
let textures: THREE.Texture[] = []
let animationId = 0
let resizeObserver: ResizeObserver | null = null
let autoPlayTimer: ReturnType<typeof setInterval> | null = null
let cleanup: (() => void) | null = null

let transitionPhase: TransitionPhase = 'idle'
let transitionProgress = 0
let transitionStart = 0
let pendingIndex: number | null = null

function easeInOutCubic(t: number): number {
  return t < 0.5 ? 4 * t * t * t : 1 - (-2 * t + 2) ** 3 / 2
}

function easeOutCubic(t: number): number {
  return 1 - (1 - t) ** 3
}

function easeInCubic(t: number): number {
  return t * t * t
}

function getGridDimensions(aspect: number) {
  const height = 5.2
  const width = height * aspect
  return { width, height, cellW: width / COLS, cellH: height / ROWS }
}

function createPieceMaterial(texture: THREE.Texture, col: number, row: number) {
  const map = texture.clone()
  map.repeat.set(1 / COLS, 1 / ROWS)
  map.offset.set(col / COLS, 1 - (row + 1) / ROWS)
  map.needsUpdate = true

  return new THREE.MeshBasicMaterial({
    map,
    side: THREE.DoubleSide,
    transparent: true,
  })
}

function buildPieces(texture: THREE.Texture, aspect: number): Piece[] {
  if (!scene) return []

  const { width, height, cellW, cellH } = getGridDimensions(aspect)
  const result: Piece[] = []

  for (let row = 0; row < ROWS; row++) {
    for (let col = 0; col < COLS; col++) {
      const geometry = new THREE.PlaneGeometry(cellW * 0.98, cellH * 0.98)
      const material = createPieceMaterial(texture, col, row)
      const mesh = new THREE.Mesh(geometry, material)

      const x = -width / 2 + cellW / 2 + col * cellW
      const y = height / 2 - cellH / 2 - row * cellH
      mesh.position.set(x, y, 0)

      const scatterPosition = new THREE.Vector3(
        x + (Math.random() - 0.5) * width * 1.4,
        y + (Math.random() - 0.5) * height * 1.4,
        (Math.random() - 0.5) * 6 + (Math.random() > 0.5 ? 2 : -2),
      )
      const scatterRotation = new THREE.Euler(
        (Math.random() - 0.5) * Math.PI * 1.2,
        (Math.random() - 0.5) * Math.PI * 1.2,
        (Math.random() - 0.5) * Math.PI * 0.8,
      )

      scene.add(mesh)
      result.push({
        mesh,
        homePosition: mesh.position.clone(),
        homeRotation: mesh.rotation.clone(),
        scatterPosition,
        scatterRotation,
      })
    }
  }

  return result
}

function disposePieces() {
  for (const piece of pieces) {
    scene?.remove(piece.mesh)
    piece.mesh.geometry.dispose()
    const material = piece.mesh.material as THREE.MeshBasicMaterial
    material.map?.dispose()
    material.dispose()
  }
  pieces = []
}

function updatePieceMaterials(texture: THREE.Texture) {
  let i = 0
  for (let row = 0; row < ROWS; row++) {
    for (let col = 0; col < COLS; col++) {
      const piece = pieces[i++]
      const material = piece.mesh.material as THREE.MeshBasicMaterial
      material.map?.dispose()
      const map = texture.clone()
      map.repeat.set(1 / COLS, 1 / ROWS)
      map.offset.set(col / COLS, 1 - (row + 1) / ROWS)
      map.needsUpdate = true
      material.map = map
      material.needsUpdate = true
    }
  }
}

function refreshScatterTargets(aspect: number) {
  const { width, height } = getGridDimensions(aspect)
  for (const piece of pieces) {
    piece.scatterPosition.set(
      piece.homePosition.x + (Math.random() - 0.5) * width * 1.4,
      piece.homePosition.y + (Math.random() - 0.5) * height * 1.4,
      (Math.random() - 0.5) * 6 + (Math.random() > 0.5 ? 2 : -2),
    )
    piece.scatterRotation.set(
      (Math.random() - 0.5) * Math.PI * 1.2,
      (Math.random() - 0.5) * Math.PI * 1.2,
      (Math.random() - 0.5) * Math.PI * 0.8,
    )
  }
}

function applyPieceState(from: 'home' | 'scatter', to: 'home' | 'scatter', t: number) {
  const eased = easeInOutCubic(t)
  for (const piece of pieces) {
    const fromPos = from === 'home' ? piece.homePosition : piece.scatterPosition
    const toPos = to === 'home' ? piece.homePosition : piece.scatterPosition
    const fromRot = from === 'home' ? piece.homeRotation : piece.scatterRotation
    const toRot = to === 'home' ? piece.homeRotation : piece.scatterRotation

    piece.mesh.position.lerpVectors(fromPos, toPos, eased)
    piece.mesh.rotation.x = THREE.MathUtils.lerp(fromRot.x, toRot.x, eased)
    piece.mesh.rotation.y = THREE.MathUtils.lerp(fromRot.y, toRot.y, eased)
    piece.mesh.rotation.z = THREE.MathUtils.lerp(fromRot.z, toRot.z, eased)
  }
}

function getAspect(container: HTMLElement): number {
  return container.clientWidth / Math.max(container.clientHeight, 1)
}

function goToSlide(index: number) {
  if (isTransitioning.value || index === currentIndex.value) return
  pendingIndex = index
  transitionPhase = 'breaking'
  transitionProgress = 0
  transitionStart = performance.now()
  isTransitioning.value = true
  refreshScatterTargets(getAspect(containerRef.value!))
}

function nextSlide() {
  goToSlide((currentIndex.value + 1) % slides.length)
}

function prevSlide() {
  goToSlide((currentIndex.value - 1 + slides.length) % slides.length)
}

function startAutoPlay() {
  autoPlayTimer = setInterval(() => {
    if (!isTransitioning.value) nextSlide()
  }, 6000)
}

onMounted(async () => {
  const container = containerRef.value
  if (!container) return

  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(45, 1, 0.1, 100)
  camera.position.z = 12

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setClearColor(0x000000, 0)
  container.appendChild(renderer.domElement)

  const loader = new THREE.TextureLoader()
  loader.setCrossOrigin('anonymous')

  textures = await Promise.all(
    slides.map(
      (slide) =>
        new Promise<THREE.Texture>((resolve, reject) => {
          loader.load(
            slide.image,
            (texture) => {
              texture.colorSpace = THREE.SRGBColorSpace
              resolve(texture)
            },
            undefined,
            reject,
          )
        }),
    ),
  )

  const aspect = getAspect(container)
  pieces = buildPieces(textures[currentIndex.value], aspect)

  const clock = new THREE.Clock()

  const resize = () => {
    const width = container.clientWidth
    const height = container.clientHeight
    if (!camera || !renderer) return
    camera.aspect = width / Math.max(height, 1)
    camera.updateProjectionMatrix()
    renderer.setSize(width, height, false)
  }

  resizeObserver = new ResizeObserver(resize)
  resizeObserver.observe(container)
  resize()

  const animate = () => {
    animationId = requestAnimationFrame(animate)
    const elapsed = clock.getElapsedTime()

    if (transitionPhase !== 'idle') {
      const now = performance.now()
      const duration =
        transitionPhase === 'breaking' ? BREAK_DURATION * 1000 : JOIN_DURATION * 1000
      transitionProgress = Math.min((now - transitionStart) / duration, 1)

      if (transitionPhase === 'breaking') {
        applyPieceState('home', 'scatter', easeOutCubic(transitionProgress))
        if (transitionProgress >= 1 && pendingIndex !== null) {
          currentIndex.value = pendingIndex
          updatePieceMaterials(textures[pendingIndex])
          transitionPhase = 'joining'
          transitionProgress = 0
          transitionStart = now
        }
      } else if (transitionPhase === 'joining') {
        applyPieceState('scatter', 'home', easeInCubic(transitionProgress))
        if (transitionProgress >= 1) {
          transitionPhase = 'idle'
          pendingIndex = null
          isTransitioning.value = false
          for (const piece of pieces) {
            piece.mesh.position.copy(piece.homePosition)
            piece.mesh.rotation.copy(piece.homeRotation)
          }
        }
      }
    } else {
      for (const piece of pieces) {
        piece.mesh.position.z = Math.sin(elapsed * 0.6 + piece.homePosition.x) * 0.03
      }
    }

    renderer?.render(scene!, camera!)
  }

  animate()
  startAutoPlay()

  const onKeyDown = (event: KeyboardEvent) => {
    if (event.key === 'ArrowRight') nextSlide()
    if (event.key === 'ArrowLeft') prevSlide()
  }
  window.addEventListener('keydown', onKeyDown)

  cleanup = () => {
    cancelAnimationFrame(animationId)
    window.removeEventListener('keydown', onKeyDown)
    resizeObserver?.disconnect()
    if (autoPlayTimer) clearInterval(autoPlayTimer)
    disposePieces()
    for (const texture of textures) texture.dispose()
    renderer?.dispose()
    if (renderer?.domElement.parentElement === container) {
      container.removeChild(renderer.domElement)
    }
    renderer = null
    scene = null
    camera = null
  }
})

onUnmounted(() => {
  cleanup?.()
  cleanup = null
})
</script>

<template>
  <section class="slider-section" aria-label="Image gallery">
    <div class="slider-header">
      <p class="slider-eyebrow">Gallery</p>
      <h2 class="slider-title">Explore the world</h2>
      <p class="slider-subtitle">
        Each slide shatters into fragments and reassembles into the next scene.
      </p>
    </div>

    <div class="slider-stage">
      <div ref="containerRef" class="slider-canvas" role="img" :aria-label="slides[currentIndex].title" />

      <div class="slider-overlay">
        <div class="slide-info">
          <span class="slide-index">{{ String(currentIndex + 1).padStart(2, '0') }}</span>
          <div>
            <h3 class="slide-title">{{ slides[currentIndex].title }}</h3>
            <p class="slide-caption">{{ slides[currentIndex].caption }}</p>
          </div>
        </div>

        <div class="slider-controls">
          <button
            type="button"
            class="nav-btn"
            aria-label="Previous slide"
            :disabled="isTransitioning"
            @click="prevSlide"
          >
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M15 18l-6-6 6-6" />
            </svg>
          </button>

          <div class="slider-dots" role="tablist" aria-label="Slide navigation">
            <button
              v-for="(slide, index) in slides"
              :key="slide.title"
              type="button"
              role="tab"
              class="dot"
              :class="{ active: index === currentIndex }"
              :aria-selected="index === currentIndex"
              :aria-label="`Go to slide ${index + 1}: ${slide.title}`"
              :disabled="isTransitioning"
              @click="goToSlide(index)"
            />
          </div>

          <button
            type="button"
            class="nav-btn"
            aria-label="Next slide"
            :disabled="isTransitioning"
            @click="nextSlide"
          >
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9 18l6-6-6-6" />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.slider-section {
  padding: 80px 24px 100px;
}

.slider-header {
  max-width: 640px;
  margin: 0 auto 48px;
  text-align: center;
}

.slider-eyebrow {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--accent);
  margin: 0 0 12px;
}

.slider-title {
  font-size: clamp(32px, 5vw, 48px);
  letter-spacing: -0.03em;
  margin: 0 0 16px;
  color: var(--text-h);
}

.slider-subtitle {
  font-size: 18px;
  line-height: 1.6;
  color: var(--text);
  margin: 0;
}

.slider-stage {
  position: relative;
  max-width: 1100px;
  margin: 0 auto;
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid var(--border);
  background: var(--surface);
  box-shadow: var(--shadow-lg);
}

.slider-canvas {
  width: 100%;
  aspect-ratio: 16 / 9;
  min-height: 280px;
}

.slider-canvas :deep(canvas) {
  display: block;
  width: 100%;
  height: 100%;
}

.slider-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 24px 28px;
  pointer-events: none;
  background: linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.45) 0%,
    transparent 35%,
    transparent 55%,
    rgba(0, 0, 0, 0.55) 100%
  );
}

.slide-info {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  pointer-events: none;
}

.slide-index {
  font-family: var(--mono);
  font-size: 14px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.7);
  padding-top: 4px;
}

.slide-title {
  font-size: 28px;
  font-weight: 500;
  color: #fff;
  margin: 0 0 4px;
  letter-spacing: -0.02em;
}

.slide-caption {
  font-size: 15px;
  color: rgba(255, 255, 255, 0.75);
  margin: 0;
}

.slider-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  pointer-events: auto;
}

.nav-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.25);
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  cursor: pointer;
  backdrop-filter: blur(8px);
  transition: background 0.2s, border-color 0.2s, transform 0.2s;
}

.nav-btn svg {
  width: 20px;
  height: 20px;
}

.nav-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.45);
  transform: scale(1.05);
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.slider-dots {
  display: flex;
  gap: 10px;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: none;
  padding: 0;
  background: rgba(255, 255, 255, 0.35);
  cursor: pointer;
  transition: background 0.25s, transform 0.25s;
}

.dot.active {
  background: #fff;
  transform: scale(1.2);
}

.dot:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.7);
}

@media (max-width: 640px) {
  .slider-section {
    padding: 56px 16px 72px;
  }

  .slider-overlay {
    padding: 16px;
  }

  .slide-title {
    font-size: 22px;
  }

  .slider-controls {
    gap: 12px;
  }
}
</style>
