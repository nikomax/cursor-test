<template>
  <section id="craft" class="craft">
    <div class="craft-inner">
      <p class="eyebrow">The craft</p>
      <h2 class="title">Pixels that remember touch</h2>
      <p class="lead">
        A small playground of WebGL tricks — liquid displacement up top, shatter
        transitions below. Built to feel alive, not decorative.
      </p>

      <ul class="list">
        <li v-for="(item, index) in items" :key="item.title">
          <span class="index">{{ String(index + 1).padStart(2, '0') }}</span>
          <div>
            <h3>{{ item.title }}</h3>
            <p>{{ item.description }}</p>
          </div>
        </li>
      </ul>
    </div>
  </section>
</template>

<script setup lang="ts">
const items = [
  {
    title: 'Liquid hero',
    description:
      'A full-bleed photo warped by a custom fragment shader — mouse pull, chromatic fringing, click ripples.',
  },
  {
    title: 'Shatter gallery',
    description:
      'Each slide breaks into hundreds of fragments, drifts through 3D space, then joins the next frame.',
  },
  {
    title: 'Magnetic cursor',
    description:
      'A soft lime trail that expands over interactive targets — tiny motion, big presence.',
  },
]
</script>

<style scoped>
.craft {
  padding: 100px 24px;
  border-top: 1px solid var(--border);
}

.craft-inner {
  max-width: 920px;
  margin: 0 auto;
}

.eyebrow {
  margin: 0 0 14px;
  font-size: 12px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: var(--accent);
}

.title {
  margin: 0 0 18px;
  font-family: var(--display);
  font-size: clamp(36px, 6vw, 56px);
  line-height: 1.05;
  letter-spacing: -0.03em;
  color: var(--text-h);
}

.lead {
  margin: 0 0 48px;
  max-width: 520px;
  font-family: var(--serif);
  font-size: 20px;
  line-height: 1.55;
  color: var(--text);
}

.list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  gap: 0;
}

.list li {
  display: grid;
  grid-template-columns: 64px 1fr;
  gap: 20px;
  padding: 28px 0;
  border-top: 1px solid var(--border);
}

.list li:last-child {
  border-bottom: 1px solid var(--border);
}

.index {
  font-family: var(--mono);
  font-size: 13px;
  color: var(--accent);
  padding-top: 6px;
}

.list h3 {
  margin: 0 0 8px;
  font-family: var(--display);
  font-size: 24px;
  font-weight: 600;
  color: var(--text-h);
}

.list p {
  margin: 0;
  font-size: 16px;
  line-height: 1.6;
  color: var(--text);
  max-width: 520px;
}

@media (max-width: 640px) {
  .list li {
    grid-template-columns: 40px 1fr;
    gap: 12px;
  }
}
</style>
