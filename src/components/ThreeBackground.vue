<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLDivElement | null>(null)

let animationId = 0
let renderer: THREE.WebGLRenderer | null = null
let resizeObserver: ResizeObserver | null = null
let cleanup: (() => void) | null = null

onMounted(() => {
  const container = containerRef.value
  if (!container) return

  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(60, 1, 0.1, 100)
  camera.position.z = 14

  renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true,
    powerPreference: 'high-performance',
  })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setClearColor(0x000000, 0)
  container.appendChild(renderer.domElement)

  const accent = new THREE.Color('#aa3bff')
  const accentSoft = new THREE.Color('#c084fc')
  const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches
  const particleColor = isDark ? accentSoft : accent

  const particleCount = 800
  const positions = new Float32Array(particleCount * 3)
  const velocities: THREE.Vector3[] = []

  for (let i = 0; i < particleCount; i++) {
    const radius = 6 + Math.random() * 10
    const theta = Math.random() * Math.PI * 2
    const phi = Math.acos(2 * Math.random() - 1)

    positions[i * 3] = radius * Math.sin(phi) * Math.cos(theta)
    positions[i * 3 + 1] = radius * Math.sin(phi) * Math.sin(theta)
    positions[i * 3 + 2] = radius * Math.cos(phi)

    velocities.push(
      new THREE.Vector3(
        (Math.random() - 0.5) * 0.008,
        (Math.random() - 0.5) * 0.008,
        (Math.random() - 0.5) * 0.008,
      ),
    )
  }

  const particleGeometry = new THREE.BufferGeometry()
  particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

  const particles = new THREE.Points(
    particleGeometry,
    new THREE.PointsMaterial({
      color: particleColor,
      size: 0.06,
      transparent: true,
      opacity: isDark ? 0.75 : 0.55,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  scene.add(particles)

  const linePositions: number[] = []
  const connectionDistanceSq = 2.8 * 2.8

  for (let i = 0; i < particleCount; i++) {
    for (let j = i + 1; j < particleCount; j++) {
      const dx = positions[i * 3] - positions[j * 3]
      const dy = positions[i * 3 + 1] - positions[j * 3 + 1]
      const dz = positions[i * 3 + 2] - positions[j * 3 + 2]
      if (dx * dx + dy * dy + dz * dz < connectionDistanceSq) {
        linePositions.push(
          positions[i * 3],
          positions[i * 3 + 1],
          positions[i * 3 + 2],
          positions[j * 3],
          positions[j * 3 + 1],
          positions[j * 3 + 2],
        )
      }
    }
  }

  const lineGeometry = new THREE.BufferGeometry()
  lineGeometry.setAttribute(
    'position',
    new THREE.Float32BufferAttribute(linePositions, 3),
  )

  const lines = new THREE.LineSegments(
    lineGeometry,
    new THREE.LineBasicMaterial({
      color: particleColor,
      transparent: true,
      opacity: isDark ? 0.12 : 0.08,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  scene.add(lines)

  const wireMaterial = new THREE.MeshBasicMaterial({
    color: particleColor,
    wireframe: true,
    transparent: true,
    opacity: isDark ? 0.35 : 0.22,
  })

  const torusKnot = new THREE.Mesh(
    new THREE.TorusKnotGeometry(2.4, 0.55, 180, 24),
    wireMaterial,
  )
  torusKnot.position.set(-4.5, 1.2, -2)
  scene.add(torusKnot)

  const icosahedron = new THREE.Mesh(
    new THREE.IcosahedronGeometry(2.1, 1),
    wireMaterial.clone(),
  )
  icosahedron.position.set(5, -1.5, -3)
  scene.add(icosahedron)

  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(3.6, 0.04, 12, 120),
    new THREE.MeshBasicMaterial({
      color: particleColor,
      transparent: true,
      opacity: isDark ? 0.4 : 0.25,
    }),
  )
  ring.rotation.x = Math.PI / 2.4
  scene.add(ring)

  const mouse = new THREE.Vector2()
  const targetMouse = new THREE.Vector2()

  const onPointerMove = (event: PointerEvent) => {
    targetMouse.x = (event.clientX / window.innerWidth) * 2 - 1
    targetMouse.y = -(event.clientY / window.innerHeight) * 2 + 1
  }

  const onThemeChange = (event: MediaQueryListEvent) => {
    const dark = event.matches
    const nextColor = dark ? accentSoft : accent
    ;(particles.material as THREE.PointsMaterial).color.copy(nextColor)
    ;(particles.material as THREE.PointsMaterial).opacity = dark ? 0.75 : 0.55
    ;(lines.material as THREE.LineBasicMaterial).color.copy(nextColor)
    ;(lines.material as THREE.LineBasicMaterial).opacity = dark ? 0.12 : 0.08
    wireMaterial.color.copy(nextColor)
    wireMaterial.opacity = dark ? 0.35 : 0.22
    ;(ring.material as THREE.MeshBasicMaterial).color.copy(nextColor)
    ;(ring.material as THREE.MeshBasicMaterial).opacity = dark ? 0.4 : 0.25
  }

  const themeQuery = window.matchMedia('(prefers-color-scheme: dark)')
  themeQuery.addEventListener('change', onThemeChange)
  window.addEventListener('pointermove', onPointerMove)

  const clock = new THREE.Clock()
  const positionAttr = particleGeometry.getAttribute('position') as THREE.BufferAttribute

  const resize = () => {
    const width = container.clientWidth
    const height = container.clientHeight
    camera.aspect = width / height
    camera.updateProjectionMatrix()
    renderer?.setSize(width, height, false)
  }

  resizeObserver = new ResizeObserver(resize)
  resizeObserver.observe(container)
  resize()

  const animate = () => {
    animationId = requestAnimationFrame(animate)
    const elapsed = clock.getElapsedTime()

    mouse.lerp(targetMouse, 0.04)
    camera.position.x = mouse.x * 1.8
    camera.position.y = mouse.y * 1.2
    camera.lookAt(0, 0, 0)

    for (let i = 0; i < particleCount; i++) {
      positions[i * 3] += velocities[i].x
      positions[i * 3 + 1] += velocities[i].y
      positions[i * 3 + 2] += velocities[i].z

      const dist = Math.hypot(
        positions[i * 3],
        positions[i * 3 + 1],
        positions[i * 3 + 2],
      )

      if (dist > 16 || dist < 4) {
        velocities[i].multiplyScalar(-1)
      }
    }

    positionAttr.needsUpdate = true
    particles.rotation.y = elapsed * 0.03
    lines.rotation.y = elapsed * 0.03

    torusKnot.rotation.x = elapsed * 0.22
    torusKnot.rotation.y = elapsed * 0.31
    icosahedron.rotation.x = elapsed * 0.18
    icosahedron.rotation.z = elapsed * 0.24
    ring.rotation.z = elapsed * 0.12

    renderer?.render(scene, camera)
  }

  animate()

  cleanup = () => {
    cancelAnimationFrame(animationId)
    themeQuery.removeEventListener('change', onThemeChange)
    window.removeEventListener('pointermove', onPointerMove)
    resizeObserver?.disconnect()
    resizeObserver = null

    particleGeometry.dispose()
    lineGeometry.dispose()
    torusKnot.geometry.dispose()
    icosahedron.geometry.dispose()
    ring.geometry.dispose()
    ;(particles.material as THREE.Material).dispose()
    ;(lines.material as THREE.Material).dispose()
    wireMaterial.dispose()
    ;(ring.material as THREE.Material).dispose()

    renderer?.dispose()
    if (renderer?.domElement.parentElement === container) {
      container.removeChild(renderer.domElement)
    }
    renderer = null
  }
})

onUnmounted(() => {
  cleanup?.()
  cleanup = null
})
</script>

<template>
  <div ref="containerRef" class="three-background" aria-hidden="true" />
</template>

<style scoped>
.three-background {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  overflow: hidden;
}

.three-background :deep(canvas) {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
