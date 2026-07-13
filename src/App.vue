<script setup lang="ts">
import HeroSection from './components/HeroSection.vue'
import ImageSlider from './components/ImageSlider.vue'
import FeaturesSection from './components/FeaturesSection.vue'
import FooterSection from './components/FooterSection.vue'
</script>

<template>
  <div class="landing">
    <HeroSection />
    <div id="gallery">
      <ImageSlider />
    </div>
    <FeaturesSection />
    <FooterSection />
  </div>
</template>
