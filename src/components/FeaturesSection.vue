<template>
  <section class="features">
    <div class="features-inner">
      <h2 class="features-title">Built for exploration</h2>
      <div class="features-grid">
        <article v-for="feature in features" :key="feature.title" class="feature-card">
          <div class="feature-icon" v-html="feature.icon" />
          <h3>{{ feature.title }}</h3>
          <p>{{ feature.description }}</p>
        </article>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
const features = [
  {
    title: 'Shatter transitions',
    description:
      'Each image breaks into a grid of fragments that scatter in 3D space before reassembling.',
    icon: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>',
  },
  {
    title: 'Curated photography',
    description:
      'Stunning landscapes sourced from Unsplash, hand-picked to showcase natural beauty.',
    icon: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><circle cx="12" cy="12" r="3"/><path d="M12 2v2M12 20v2M2 12h2M20 12h2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41"/></svg>',
  },
  {
    title: 'Smooth performance',
    description:
      'Hardware-accelerated WebGL rendering with optimized geometry for fluid 60fps animations.',
    icon: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/></svg>',
  },
]
</script>

<style scoped>
.features {
  padding: 80px 24px;
  border-top: 1px solid var(--border);
}

.features-inner {
  max-width: 1100px;
  margin: 0 auto;
}

.features-title {
  font-size: clamp(28px, 4vw, 36px);
  text-align: center;
  letter-spacing: -0.02em;
  margin: 0 0 48px;
  color: var(--text-h);
}

.features-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.feature-card {
  padding: 32px 28px;
  border-radius: 16px;
  border: 1px solid var(--border);
  background: var(--surface);
  transition: border-color 0.2s, transform 0.2s;
}

.feature-card:hover {
  border-color: var(--accent-border);
  transform: translateY(-4px);
}

.feature-icon {
  width: 40px;
  height: 40px;
  color: var(--accent);
  margin-bottom: 20px;
}

.feature-icon :deep(svg) {
  width: 100%;
  height: 100%;
}

.feature-card h3 {
  font-size: 20px;
  font-weight: 500;
  color: var(--text-h);
  margin: 0 0 10px;
}

.feature-card p {
  font-size: 15px;
  line-height: 1.6;
  color: var(--text);
  margin: 0;
}

@media (max-width: 768px) {
  .features-grid {
    grid-template-columns: 1fr;
  }
}
</style>
