<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLDivElement | null>(null)
const brandVisible = ref(false)

const HERO_IMAGE =
  'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=1920&q=85'

const vertexShader = /* glsl */ `
  varying vec2 vUv;
  void main() {
    vUv = uv;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
`

const fragmentShader = /* glsl */ `
  uniform sampler2D uTexture;
  uniform vec2 uMouse;
  uniform vec2 uResolution;
  uniform float uTime;
  uniform float uHover;
  uniform float uRipple;
  uniform vec2 uRippleOrigin;

  varying vec2 vUv;

  float hash(vec2 p) {
    return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453);
  }

  void main() {
    vec2 uv = vUv;

    float wave = sin(uv.y * 8.0 + uTime * 0.7) * 0.004
               + cos(uv.x * 6.0 - uTime * 0.5) * 0.003;
    uv += wave;

    vec2 mouseUv = uMouse;
    float dist = distance(uv, mouseUv);
    float influence = smoothstep(0.45, 0.0, dist) * uHover;
    vec2 dir = normalize(uv - mouseUv + 0.0001);
    uv += dir * influence * 0.06;
    uv += vec2(
      sin(dist * 28.0 - uTime * 3.0),
      cos(dist * 24.0 - uTime * 2.4)
    ) * influence * 0.012;

    float rDist = distance(uv, uRippleOrigin);
    float rippleWave = sin(rDist * 40.0 - uRipple * 12.0) * exp(-rDist * 4.0) * (1.0 - uRipple);
    uv += normalize(uv - uRippleOrigin + 0.0001) * rippleWave * 0.03;

    float imgAspect = 1.777;
    float screenAspect = uResolution.x / max(uResolution.y, 1.0);
    vec2 coverUv = uv;
    if (screenAspect > imgAspect) {
      float scale = imgAspect / screenAspect;
      coverUv.y = (uv.y - 0.5) * scale + 0.5;
    } else {
      float scale = screenAspect / imgAspect;
      coverUv.x = (uv.x - 0.5) * scale + 0.5;
    }

    float chroma = influence * 0.008;
    float r = texture2D(uTexture, coverUv + vec2(chroma, 0.0)).r;
    float g = texture2D(uTexture, coverUv).g;
    float b = texture2D(uTexture, coverUv - vec2(chroma, 0.0)).b;
    vec3 color = vec3(r, g, b);

    float grain = (hash(uv * uTime) - 0.5) * 0.045;
    color += grain;

    float vignette = smoothstep(1.15, 0.35, length(vUv - 0.5));
    color *= mix(0.55, 1.0, vignette);
    color = mix(color, color * vec3(0.92, 1.05, 1.08), 0.18);

    gl_FragColor = vec4(color, 1.0);
  }
`

let triggerRipple: ((clientX: number, clientY: number) => void) | null = null
let cleanup: (() => void) | null = null

function stirWater() {
  triggerRipple?.(window.innerWidth * 0.5, window.innerHeight * 0.42)
}

onMounted(() => {
  const container = containerRef.value
  if (!container) return

  requestAnimationFrame(() => {
    brandVisible.value = true
  })

  const scene = new THREE.Scene()
  const camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0.1, 10)
  camera.position.z = 1

  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setClearColor(0x0a1210, 1)
  container.appendChild(renderer.domElement)

  const mouse = new THREE.Vector2(0.5, 0.5)
  const targetMouse = new THREE.Vector2(0.5, 0.5)
  let hover = 0
  let targetHover = 0
  let ripple = 1
  const rippleOrigin = new THREE.Vector2(0.5, 0.5)

  const placeholder = new THREE.DataTexture(new Uint8Array([10, 18, 16, 255]), 1, 1)
  placeholder.needsUpdate = true

  const uniforms = {
    uTexture: { value: placeholder as THREE.Texture },
    uMouse: { value: mouse },
    uResolution: { value: new THREE.Vector2(1, 1) },
    uTime: { value: 0 },
    uHover: { value: 0 },
    uRipple: { value: 1 },
    uRippleOrigin: { value: rippleOrigin },
  }

  const geometry = new THREE.PlaneGeometry(2, 2)
  const material = new THREE.ShaderMaterial({
    uniforms,
    vertexShader,
    fragmentShader,
  })
  scene.add(new THREE.Mesh(geometry, material))

  const loader = new THREE.TextureLoader()
  loader.setCrossOrigin('anonymous')
  loader.load(HERO_IMAGE, (texture) => {
    texture.colorSpace = THREE.SRGBColorSpace
    texture.minFilter = THREE.LinearFilter
    texture.magFilter = THREE.LinearFilter
    uniforms.uTexture.value = texture
  })

  const fireRipple = (clientX: number, clientY: number) => {
    const rect = container.getBoundingClientRect()
    rippleOrigin.x = (clientX - rect.left) / Math.max(rect.width, 1)
    rippleOrigin.y = 1 - (clientY - rect.top) / Math.max(rect.height, 1)
    ripple = 0
  }
  triggerRipple = fireRipple

  const resize = () => {
    const width = container.clientWidth
    const height = container.clientHeight
    renderer.setSize(width, height, false)
    uniforms.uResolution.value.set(width, height)
  }

  const resizeObserver = new ResizeObserver(resize)
  resizeObserver.observe(container)
  resize()

  const onPointerMove = (event: PointerEvent) => {
    const rect = container.getBoundingClientRect()
    targetMouse.x = (event.clientX - rect.left) / Math.max(rect.width, 1)
    targetMouse.y = 1 - (event.clientY - rect.top) / Math.max(rect.height, 1)
    targetHover = 1
  }

  const onPointerLeave = () => {
    targetHover = 0
  }

  const onPointerDown = (event: PointerEvent) => {
    fireRipple(event.clientX, event.clientY)
  }

  container.addEventListener('pointermove', onPointerMove)
  container.addEventListener('pointerleave', onPointerLeave)
  container.addEventListener('pointerdown', onPointerDown)

  let animationId = 0
  const clock = new THREE.Clock()

  const animate = () => {
    animationId = requestAnimationFrame(animate)
    uniforms.uTime.value = clock.getElapsedTime()
    mouse.lerp(targetMouse, 0.06)
    hover += (targetHover - hover) * 0.05
    uniforms.uHover.value = hover
    if (ripple < 1) ripple = Math.min(ripple + 0.012, 1)
    uniforms.uRipple.value = ripple
    renderer.render(scene, camera)
  }
  animate()

  cleanup = () => {
    cancelAnimationFrame(animationId)
    resizeObserver.disconnect()
    container.removeEventListener('pointermove', onPointerMove)
    container.removeEventListener('pointerleave', onPointerLeave)
    container.removeEventListener('pointerdown', onPointerDown)
    triggerRipple = null
    geometry.dispose()
    material.dispose()
    const currentTexture = uniforms.uTexture.value
    if (currentTexture && currentTexture !== placeholder) {
      currentTexture.dispose()
    }
    placeholder.dispose()
    renderer.dispose()
    if (renderer.domElement.parentElement === container) {
      container.removeChild(renderer.domElement)
    }
  }
})

onUnmounted(() => {
  cleanup?.()
  cleanup = null
})
</script>

<template>
  <section class="hero" aria-label="DRIFT hero">
    <div ref="containerRef" class="hero-canvas" />
    <div class="hero-veil" aria-hidden="true" />

    <header class="topbar">
      <a class="logo" href="#">DRIFT</a>
      <nav class="nav">
        <a href="#gallery">Gallery</a>
        <a href="#craft">Craft</a>
      </nav>
    </header>

    <div class="hero-copy" :class="{ visible: brandVisible }">
      <p class="eyebrow">Interactive landscapes</p>
      <h1 class="brand">DRIFT</h1>
      <p class="tagline">
        Move through light. Touch the frame. Watch the world bend.
      </p>
      <div class="cta-row">
        <a href="#gallery" class="cta primary">Enter gallery</a>
        <button type="button" class="cta ghost" @click="stirWater">
          Stir the water
        </button>
      </div>
    </div>

    <div class="hint" aria-hidden="true">
      <span>Drag to warp · Click to ripple</span>
    </div>
  </section>
</template>

<style scoped>
.hero {
  position: relative;
  min-height: 100svh;
  overflow: hidden;
  color: #f4f0e6;
}

.hero-canvas {
  position: absolute;
  inset: 0;
}

.hero-canvas :deep(canvas) {
  display: block;
  width: 100%;
  height: 100%;
}

.hero-veil {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background:
    radial-gradient(ellipse 70% 55% at 50% 42%, rgba(8, 18, 16, 0.15), rgba(8, 18, 16, 0.72) 70%),
    linear-gradient(180deg, rgba(8, 18, 16, 0.35) 0%, transparent 28%, transparent 62%, rgba(8, 18, 16, 0.85) 100%);
}

.topbar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 28px clamp(20px, 4vw, 48px);
}

.logo {
  font-family: var(--display);
  font-size: 22px;
  font-weight: 700;
  letter-spacing: 0.18em;
  text-decoration: none;
  color: #f4f0e6;
}

.nav {
  display: flex;
  gap: 28px;
}

.nav a {
  color: rgba(244, 240, 230, 0.75);
  text-decoration: none;
  font-size: 13px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  transition: color 0.2s;
}

.nav a:hover {
  color: #d6ff4b;
}

.hero-copy {
  position: relative;
  z-index: 2;
  min-height: 100svh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 120px 24px 100px;
  pointer-events: none;
  opacity: 0;
  transform: translateY(28px);
  transition:
    opacity 1.1s cubic-bezier(0.22, 1, 0.36, 1),
    transform 1.1s cubic-bezier(0.22, 1, 0.36, 1);
}

.hero-copy.visible {
  opacity: 1;
  transform: translateY(0);
}

.eyebrow {
  margin: 0 0 18px;
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: #d6ff4b;
}

.brand {
  margin: 0;
  font-family: var(--display);
  font-size: clamp(72px, 18vw, 168px);
  font-weight: 700;
  line-height: 0.85;
  letter-spacing: -0.04em;
  color: #f4f0e6;
  text-shadow: 0 10px 40px rgba(0, 0, 0, 0.35);
}

.tagline {
  margin: 28px 0 0;
  max-width: 420px;
  font-family: var(--serif);
  font-size: clamp(18px, 2.4vw, 22px);
  font-weight: 400;
  line-height: 1.45;
  color: rgba(244, 240, 230, 0.82);
}

.cta-row {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
  margin-top: 36px;
  pointer-events: auto;
}

.cta {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 48px;
  padding: 0 24px;
  border-radius: 999px;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.04em;
  text-decoration: none;
  border: 1px solid transparent;
  cursor: pointer;
  transition:
    transform 0.25s ease,
    background 0.25s ease,
    border-color 0.25s ease;
}

.cta.primary {
  background: #d6ff4b;
  color: #0a1210;
}

.cta.primary:hover {
  transform: translateY(-2px);
  background: #e4ff7a;
}

.cta.ghost {
  background: rgba(244, 240, 230, 0.08);
  border-color: rgba(244, 240, 230, 0.28);
  color: #f4f0e6;
  backdrop-filter: blur(8px);
}

.cta.ghost:hover {
  border-color: rgba(214, 255, 75, 0.55);
  color: #d6ff4b;
}

.hint {
  position: absolute;
  bottom: 36px;
  left: 0;
  right: 0;
  z-index: 2;
  text-align: center;
  font-size: 11px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: rgba(244, 240, 230, 0.45);
  pointer-events: none;
  animation: hint-fade 2.4s ease-in-out infinite;
}

@keyframes hint-fade {
  0%,
  100% {
    opacity: 0.4;
  }
  50% {
    opacity: 0.9;
  }
}

@media (max-width: 640px) {
  .topbar {
    padding: 20px 18px;
  }

  .nav {
    gap: 16px;
  }

  .hint {
    display: none;
  }
}
</style>
