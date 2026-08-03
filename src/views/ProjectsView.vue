<template>
  <main class="projects-page">
    <TopBar />

    <div class="page-content">
      <!-- Hero Header -->
      <div class="page-header">
        <span class="section-kicker">🚀 Projects</span>
        <h1>Things I've Built</h1>
        <p class="page-desc">
          A collection of projects ranging from scalable backend systems to
          interactive web applications. Each one taught me something new.
        </p>
      </div>

      <!-- Loading -->
      <div v-if="loading" class="loading-state">
        <div class="skeleton-grid">
          <div v-for="n in 6" :key="n" class="skeleton-card">
            <div class="skeleton-img"></div>
            <div class="skeleton-body">
              <div class="skeleton-line w80"></div>
              <div class="skeleton-line w60"></div>
              <div class="skeleton-line w40"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty -->
      <div v-else-if="projects.length === 0" class="empty-state">
        <div class="empty-icon">🔧</div>
        <h3>No projects yet</h3>
        <p>Check back soon — something is always being built.</p>
      </div>

      <!-- Grid -->
      <div v-else class="projects-grid">
        <div
          v-for="project in projects"
          :key="project.id"
          class="project-card"
          :class="{ featured: project.featured }"
        >
          <!-- Featured badge -->
          <div v-if="project.featured" class="featured-badge">⭐ Featured</div>

          <div class="card-cover">
            <img
              v-if="project.cover_image_url"
              :src="project.cover_image_url"
              :alt="project.title"
              loading="lazy"
              decoding="async"
            />
            <div v-else class="cover-fallback" :style="fallbackStyle(project)">
              <span class="blob blob-a"></span>
              <span class="blob blob-b"></span>
              <svg class="fallback-icon" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <rect x="2" y="3" width="20" height="14" rx="2"/>
                <path d="M8 21h8M12 17v4"/>
              </svg>
            </div>
            <div class="cover-scrim"></div>
          </div>

          <div class="card-body">
            <h2 class="card-title">{{ project.title }}</h2>
            <p class="card-desc">{{ project.description }}</p>

            <!-- Tech Stack -->
            <div v-if="project.tech_stack?.length" class="tech-stack">
              <span
                v-for="tech in project.tech_stack.slice(0, 5)"
                :key="tech"
                class="tech-badge"
              >{{ tech }}</span>
            </div>

            <!-- Links -->
            <div class="card-links">
              <a
                v-if="project.live_url"
                :href="project.live_url"
                target="_blank"
                rel="noopener noreferrer"
                class="card-link live"
                @click.stop
              >
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15 3 21 3 21 9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
                Live Demo
              </a>
              <a
                v-if="project.github_url"
                :href="project.github_url"
                target="_blank"
                rel="noopener noreferrer"
                class="card-link github"
                @click.stop
              >
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
                </svg>
                GitHub
              </a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import TopBar from '../components/TopBar.vue'
import { supabase } from '../lib/supabase'

interface Project {
  id: string
  title: string
  description: string | null
  cover_image_url: string | null
  tech_stack: string[] | null
  live_url: string | null
  github_url: string | null
  featured: boolean
  order_index: number
  created_at: string
}

const projects = ref<Project[]>([])
const loading = ref(true)

const GRADIENTS = [
  ['#6c63ff', '#a78bfa'],
  ['#f472b6', '#fb923c'],
  ['#22d3ee', '#6366f1'],
  ['#34d399', '#3b82f6'],
  ['#f59e0b', '#ef4444'],
  ['#8b5cf6', '#ec4899'],
]

function fallbackStyle(project: Project) {
  const i = projects.value.findIndex(p => p.id === project.id)
  const [a, b] = GRADIENTS[(i >= 0 ? i : 0) % GRADIENTS.length]
  return { '--grad-a': a, '--grad-b': b } as Record<string, string>
}

onMounted(async () => {
  const { data, error } = await supabase
    .from('projects')
    .select('id, title, description, cover_image_url, tech_stack, live_url, github_url, featured, order_index, created_at')
    .eq('published', true)
    .order('order_index', { ascending: true })
  if (!error) projects.value = data ?? []
  loading.value = false
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Inter:wght@300;400;500;600&display=swap');

.projects-page {
  min-height: 100vh;
  background: #ffffff;
  font-family: 'Inter', sans-serif;
}

.page-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 120px 40px 100px;
}

/* ── Header ── */
.page-header {
  text-align: center;
  margin-bottom: 64px;
}

.section-kicker {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  font-weight: 700;
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.08);
  border: 1px solid rgba(108, 99, 255, 0.2);
  padding: 5px 14px;
  border-radius: 100px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  margin-bottom: 20px;
}

.page-header h1 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(32px, 5vw, 56px);
  font-weight: 700;
  color: #1a1a2e;
  letter-spacing: -1.5px;
  margin: 0 0 16px;
  background: linear-gradient(135deg, #6c63ff 0%, #a78bfa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.page-desc {
  font-size: clamp(15px, 1.4vw, 17px);
  color: #64748b;
  line-height: 1.7;
  max-width: 560px;
  margin: 0 auto;
}

/* ── Projects Grid ── */
.projects-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 28px;
}

/* ── Card ── */
.project-card {
  display: flex;
  flex-direction: column;
  background: #fff;
  border: 1px solid rgba(108, 99, 255, 0.14);
  border-radius: 20px;
  overflow: hidden;
  transition: transform 0.25s ease, border-color 0.25s ease;
  position: relative;
}

.project-card::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.25s ease;
  background: linear-gradient(135deg, rgba(108, 99, 255, 0.06) 0%, transparent 55%);
}

.project-card:hover {
  transform: translateY(-4px);
  border-color: rgba(108, 99, 255, 0.38);
}

.project-card:hover::before {
  opacity: 1;
}

.project-card.featured {
  border-color: rgba(108, 99, 255, 0.32);
  background: linear-gradient(180deg, rgba(108, 99, 255, 0.04) 0%, #fff 48%);
}

.project-card.featured::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #6c63ff, #a78bfa, #6c63ff);
  z-index: 2;
}

.featured-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  font-size: 11px;
  font-weight: 700;
  color: #78350f;
  background: rgba(251, 191, 36, 0.92);
  border: 1px solid rgba(245, 158, 11, 0.45);
  padding: 4px 10px;
  border-radius: 999px;
  letter-spacing: 0.04em;
  z-index: 3;
}

.card-cover {
  width: 100%;
  aspect-ratio: 16 / 9;
  overflow: hidden;
  background: #12122a;
  flex-shrink: 0;
  position: relative;
  isolation: isolate;
}

.card-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center 22%;
  transition: transform 0.4s ease;
  display: block;
}

.project-card:hover .card-cover img {
  transform: scale(1.04);
}

.cover-fallback {
  --grad-a: #6c63ff;
  --grad-b: #a78bfa;
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, var(--grad-a) 0%, var(--grad-b) 100%);
  overflow: hidden;
}

.cover-fallback .blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(26px);
  opacity: 0.55;
  mix-blend-mode: overlay;
}

.cover-fallback .blob-a {
  width: 140px;
  height: 140px;
  background: rgba(255, 255, 255, 0.22);
  top: -30px;
  left: -20px;
}

.cover-fallback .blob-b {
  width: 110px;
  height: 110px;
  background: rgba(0, 0, 0, 0.28);
  bottom: -30px;
  right: -10px;
}

.fallback-icon {
  position: relative;
  z-index: 1;
  color: rgba(255, 255, 255, 0.85);
}

.cover-scrim {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(0, 0, 0, 0) 62%, rgba(15, 15, 35, 0.42) 100%);
  pointer-events: none;
}

.card-body {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 22px 24px 24px;
  flex: 1;
}

.card-title {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #1a1a2e;
  line-height: 1.3;
  letter-spacing: -0.3px;
  margin: 0;
}

.card-desc {
  font-size: 14px;
  line-height: 1.7;
  color: #64748b;
  margin: 0;
  flex: 1;
}

/* Tech Stack */
.tech-stack {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 4px;
}

.tech-badge {
  font-size: 11px;
  font-weight: 700;
  color: #312e81;
  background: rgba(108, 99, 255, 0.07);
  border: 1px solid rgba(108, 99, 255, 0.15);
  padding: 3px 10px;
  border-radius: 999px;
  letter-spacing: 0.03em;
}

/* Links */
.card-links {
  display: flex;
  gap: 8px;
  margin-top: 6px;
}

.card-link {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 13px;
  font-weight: 600;
  text-decoration: none;
  padding: 6px 14px;
  border-radius: 10px;
  border: 1.5px solid;
  transition: all 0.2s;
}

.card-link.live {
  color: #6c63ff;
  border-color: rgba(108,99,255,0.25);
  background: rgba(108,99,255,0.05);
}
.card-link.live:hover {
  background: rgba(108,99,255,0.12);
  border-color: #6c63ff;
}

.card-link.github {
  color: #1a1a2e;
  border-color: rgba(26,26,46,0.15);
  background: rgba(26,26,46,0.03);
}
.card-link.github:hover {
  background: rgba(26,26,46,0.08);
  border-color: rgba(26,26,46,0.3);
}

/* ── Loading Skeletons ── */
.loading-state { width: 100%; }

.skeleton-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 28px;
}

.skeleton-card {
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid #f1f5f9;
  background: #fff;
}

.skeleton-img {
  width: 100%;
  aspect-ratio: 16 / 9;
  background: linear-gradient(90deg, #f1f5f9 25%, #e2e8f0 50%, #f1f5f9 75%);
  background-size: 400% 100%;
  animation: shimmer 1.5s ease infinite;
}

.skeleton-body {
  padding: 22px 24px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.skeleton-line {
  height: 14px;
  border-radius: 6px;
  background: linear-gradient(90deg, #f1f5f9 25%, #e2e8f0 50%, #f1f5f9 75%);
  background-size: 400% 100%;
  animation: shimmer 1.5s ease infinite;
}
.skeleton-line.w80 { width: 80%; }
.skeleton-line.w60 { width: 60%; }
.skeleton-line.w40 { width: 40%; }

@keyframes shimmer {
  0%   { background-position: 100% 0; }
  100% { background-position: -100% 0; }
}

/* ── Empty ── */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 80px 0;
  text-align: center;
  color: #94a3b8;
}
.empty-icon { font-size: 3rem; }
.empty-state h3 { font-size: 1.1rem; color: #475569; margin: 0; }
.empty-state p  { font-size: 0.9rem; margin: 0; }

/* ── Responsive ── */
@media (max-width: 960px) {
  .page-content { padding: 100px 32px 80px; }
  .projects-grid, .skeleton-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 640px) {
  .page-content { padding: 80px 16px 60px; }
  .projects-grid, .skeleton-grid { grid-template-columns: 1fr; gap: 20px; }
  .page-header { margin-bottom: 40px; }

  .project-card { border-radius: 16px; }
  .project-card:hover { transform: none; }
  .project-card:hover .card-cover img { transform: none; }

  .card-cover { aspect-ratio: 16 / 10; }
  .card-cover img { object-position: center center; }

  .card-body { padding: 18px 18px 20px; }
  .card-title { font-size: 17px; }
}
</style>
