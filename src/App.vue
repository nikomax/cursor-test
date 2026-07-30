<script setup lang="ts">
import LiquidHero from './components/LiquidHero.vue'
import ImageSlider from './components/ImageSlider.vue'
import CraftSection from './components/CraftSection.vue'
import FooterSection from './components/FooterSection.vue'
import MagneticCursor from './components/MagneticCursor.vue'
</script>

<template>
  <MagneticCursor />
  <div class="landing">
    <LiquidHero />
    <div id="gallery">
      <ImageSlider />
    </div>
    <CraftSection />
    <FooterSection />
  </div>
</template>
