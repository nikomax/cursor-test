<template>
  <section class="hero">
    <div class="hero-content">
      <p class="hero-eyebrow">Welcome</p>
      <h1 class="hero-title">
        Discover places<br />
        <span class="hero-accent">beyond imagination</span>
      </h1>
      <p class="hero-description">
        A curated journey through breathtaking landscapes. Scroll down to experience
        our interactive gallery powered by Three.js.
      </p>
      <a href="#gallery" class="hero-cta">View gallery</a>
    </div>
    <div class="hero-scroll" aria-hidden="true">
      <span>Scroll</span>
      <div class="scroll-line" />
    </div>
  </section>
</template>

<style scoped>
.hero {
  position: relative;
  min-height: 100svh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 120px 24px 80px;
  text-align: center;
}

.hero-content {
  max-width: 720px;
}

.hero-eyebrow {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--accent);
  margin: 0 0 20px;
}

.hero-title {
  font-size: clamp(40px, 7vw, 72px);
  font-weight: 500;
  line-height: 1.05;
  letter-spacing: -0.03em;
  color: var(--text-h);
  margin: 0 0 24px;
}

.hero-accent {
  background: linear-gradient(135deg, var(--accent), #6366f1);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero-description {
  font-size: clamp(17px, 2.5vw, 20px);
  line-height: 1.65;
  color: var(--text);
  margin: 0 0 36px;
  max-width: 520px;
  margin-inline: auto;
}

.hero-cta {
  display: inline-flex;
  align-items: center;
  padding: 14px 32px;
  border-radius: 999px;
  background: var(--accent);
  color: #fff;
  font-size: 16px;
  font-weight: 500;
  text-decoration: none;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 24px var(--accent-glow);
}

.hero-cta:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 32px var(--accent-glow);
}

.hero-scroll {
  position: absolute;
  bottom: 40px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  color: var(--text);
  font-size: 12px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  opacity: 0.6;
}

.scroll-line {
  width: 1px;
  height: 48px;
  background: linear-gradient(180deg, var(--accent), transparent);
  animation: scroll-pulse 2s ease-in-out infinite;
}

@keyframes scroll-pulse {
  0%,
  100% {
    opacity: 0.4;
    transform: scaleY(0.8);
  }
  50% {
    opacity: 1;
    transform: scaleY(1);
  }
}
</style>
