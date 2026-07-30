<template>
  <footer class="footer">
    <div class="footer-brand">DRIFT</div>
    <p class="footer-text">
      Photos from
      <a href="https://unsplash.com" target="_blank" rel="noopener noreferrer">Unsplash</a>.
      Built with Vue &amp; Three.js — touch something.
    </p>
  </footer>
</template>

<style scoped>
.footer {
  padding: 56px 24px 48px;
  border-top: 1px solid var(--border);
  text-align: center;
}

.footer-brand {
  font-family: var(--display);
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.22em;
  color: var(--text-h);
  margin-bottom: 14px;
}

.footer-text {
  font-size: 14px;
  color: var(--text);
  margin: 0;
}

.footer-text a {
  color: var(--accent);
  text-decoration: none;
}

.footer-text a:hover {
  text-decoration: underline;
}
</style>
