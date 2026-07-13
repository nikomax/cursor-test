<template>
  <footer class="footer">
    <p class="footer-text">
      Photos from
      <a href="https://unsplash.com" target="_blank" rel="noopener noreferrer">Unsplash</a>.
      Built with Vue &amp; Three.js.
    </p>
  </footer>
</template>

<style scoped>
.footer {
  padding: 40px 24px;
  border-top: 1px solid var(--border);
  text-align: center;
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
