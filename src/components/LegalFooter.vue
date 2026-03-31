<template>
  <footer class="legal-footer">
    <div class="footer-container">
      <p class="disclaimer">
        This is an estimate based on general formulas and does not constitute legal advice. 
        Each case is unique, and past results do not guarantee future outcomes.
      </p>
      <p style="font-size: 0.6rem; color: #444; margin-top: 2rem; letter-spacing: 0.1em;">
        © {{ new Date().getFullYear() }} PERSONAL INJURY ESTIMATOR | INSPIRED BY 8 LIMBS.
      </p>
    </div>
  </footer>
</template>

<style scoped>
.disclaimer {
  font-size: 0.75rem;
  color: var(--text-secondary);
  max-width: 600px;
  margin: 0 auto;
  line-height: 1.5;
  opacity: 0.6;
}

.footer-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 4rem 1rem 6rem 1rem;
  border-top: 1px solid var(--border-color);
  text-align: center;
}

@media (max-width: 768px) {
  .footer-container {
    padding: 3rem 1rem 8rem 1rem;
  }
}
</style>
