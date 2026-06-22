<script setup lang="ts">
import TopBar from './components/TopBar.vue';
import { supabase } from './lib/supabase'
import { watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

watch(() => route.path, async (newPath) => {
  await supabase
      .from('page_views')
      .insert({ page: newPath })
}, { immediate: true })
</script>

<template>
  <div id="portfolio-root">
    <TopBar />
    <RouterView />
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  background: #ffffff;
}

#portfolio-root {
  min-height: 100vh;
  background: #ffffff;
}
</style>
