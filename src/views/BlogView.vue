<template>
  <main class="blog-page">
    <h1>Blog</h1>
  </main>
</template>

<script setup lang="ts">
// Blog page — content coming soon
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&display=swap');

.blog-page {
  min-height: 100vh;
  background: #ffffff;
  padding-top: 64px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding-left: 80px;
  padding-right: 80px;
  box-sizing: border-box;
}

h1 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(36px, 5vw, 64px);
  font-weight: 700;
  color: #1a1a2e;
  letter-spacing: -2px;
  margin-top: 64px;
  margin-bottom: 0;
  background: linear-gradient(135deg, #6c63ff 0%, #a78bfa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

@media (max-width: 900px) {
  .blog-page {
    padding-left: 40px;
    padding-right: 40px;
  }

  h1 {
    margin-top: 48px;
  }
}

@media (max-width: 640px) {
  .blog-page {
    padding-top: 56px;
    padding-left: 24px;
    padding-right: 24px;
  }

  h1 {
    margin-top: 36px;
    letter-spacing: -1px;
  }
}
</style>
