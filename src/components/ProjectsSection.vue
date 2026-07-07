<template>
  <section
      ref="sectionRef"
      class="projects-section"
      :class="{ 'is-visible': isVisible }"
      id="projects"
      aria-labelledby="projects-title"
  >
    <div class="section-inner">

      <!-- Section header -->
      <div class="section-header reveal-item">
        <div class="header-left">
          <span class="section-kicker">🚀 Projects</span>
          <h2 id="projects-title">Things I've Built</h2>
          <p class="section-desc">
            A selection of projects that pushed my skills in system design,
            backend architecture, and product thinking.
          </p>
        </div>
        <router-link to="/projects" class="view-all-btn" id="view-all-projects-btn">
          View All Projects
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <line x1="5" y1="12" x2="19" y2="12"/>
            <polyline points="12 5 19 12 12 19"/>
          </svg>
        </router-link>
      </div>

      <!-- Loading skeletons -->
      <div v-if="loading" class="carousel-wrapper">
        <div class="carousel-track no-transition">
          <div v-for="n in 3" :key="n" class="project-card skeleton-card">
            <div class="skeleton-img"></div>
            <div class="skeleton-body">
              <div class="skeleton-line w70"></div>
              <div class="skeleton-line w50"></div>
              <div class="skeleton-line w30"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty state -->
      <div v-else-if="projects.length === 0" class="empty-state reveal-item">
        <span>🔧</span>
        <p>Projects coming soon — always building something.</p>
      </div>

      <!-- Carousel -->
      <template v-else>
        <div class="carousel-wrapper reveal-item">
          <!-- Prev button -->
          <button
              class="carousel-btn prev"
              :disabled="atStart"
              @click="slide(-1)"
              aria-label="Previous projects"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <polyline points="15 18 9 12 15 6"/>
            </svg>
          </button>

          <!-- Track -->
          <div
              class="carousel-overflow"
              ref="carouselOverflow"
              @touchstart="onTouchStart"
              @touchmove="onTouchMove"
              @touchend="onTouchEnd"
          >
            <div
                class="carousel-track"
                :class="{ 'no-transition': isDragging }"
                :style="{ transform: `translateX(${trackOffset + dragDelta}px)` }"
            >
              <div
                  v-for="(project, i) in projects"
                  :key="project.id"
                  class="project-card"
                  :class="{ featured: project.featured }"
                  :ref="i === 0 ? setFirstCardRef : undefined"
                  :style="{ transitionDelay: isVisible ? `${i * 0.06}s` : '0s' }"
              >
                <!-- Featured badge -->
                <div v-if="project.featured" class="featured-badge">
                  <svg width="11" height="11" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l2.9 6.6 7.1.6-5.4 4.7 1.6 7-6.2-3.8L6 21l1.6-7-5.4-4.7 7.1-.6z"/></svg>
                  Featured
                </div>

                <!-- Cover / media -->
                <div class="card-cover">
                  <img
                      v-if="project.cover_image_url"
                      :src="project.cover_image_url"
                      :alt="project.title"
                      loading="lazy"
                  />
                  <div v-else class="cover-fallback" :style="fallbackStyle(i)">
                    <span class="blob blob-a"></span>
                    <span class="blob blob-b"></span>
                    <span class="blob blob-c"></span>
                    <svg class="fallback-icon" width="34" height="34" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6">
                      <rect x="2" y="3" width="20" height="14" rx="2"/>
                      <path d="M8 21h8M12 17v4"/>
                    </svg>
                  </div>

                  <!-- gradient scrim for legibility / polish -->
                  <div class="cover-scrim"></div>

                  <!-- quick-action overlay on hover -->
                  <div class="cover-overlay">
                    <a
                        v-if="project.live_url"
                        :href="project.live_url"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="overlay-btn"
                        aria-label="Open live site"
                        @click.stop
                    >
                      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                        <polyline points="15 3 21 3 21 9"/>
                        <line x1="10" y1="14" x2="21" y2="3"/>
                      </svg>
                    </a>
                    <a
                        v-if="project.github_url"
                        :href="project.github_url"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="overlay-btn"
                        aria-label="Open source code"
                        @click.stop
                    >
                      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
                      </svg>
                    </a>
                  </div>
                </div>

                <!-- Body -->
                <div class="card-body">
                  <h3 class="card-title">{{ project.title }}</h3>
                  <p class="card-desc">{{ project.description }}</p>

                  <!-- Tech badges -->
                  <div v-if="project.tech_stack?.length" class="tech-stack">
                    <span
                        v-for="tech in project.tech_stack.slice(0, 4)"
                        :key="tech"
                        class="tech-badge"
                    >{{ tech }}</span>
                  </div>

                  <!-- Card footer -->
                  <div class="card-footer">
                    <a
                        v-if="project.live_url"
                        :href="project.live_url"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="card-link live"
                    >
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                        <polyline points="15 3 21 3 21 9"/>
                        <line x1="10" y1="14" x2="21" y2="3"/>
                      </svg>
                      Live
                    </a>
                    <a
                        v-if="project.github_url"
                        :href="project.github_url"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="card-link github"
                    >
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
                      </svg>
                      GitHub
                    </a>
                    <router-link to="/projects" class="card-link detail">
                      All Projects →
                    </router-link>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Next button -->
          <button
              class="carousel-btn next"
              :disabled="atEnd"
              @click="slide(1)"
              aria-label="Next projects"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <polyline points="9 18 15 12 9 6"/>
            </svg>
          </button>
        </div>

        <!-- Dot indicators -->
        <div class="carousel-dots" aria-hidden="true">
          <button
              v-for="(_, i) in dotCount"
              :key="i"
              class="dot"
              :class="{ active: currentDot === i }"
              @click="goToDot(i)"
          ></button>
        </div>
      </template>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
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
}

// ── Data ──────────────────────────────────────────────────────
const projects = ref<Project[]>([])
const loading  = ref(true)

// A rotating set of tasteful gradient pairs used for the "no image" cover art
const GRADIENTS = [
  ['#6c63ff', '#a78bfa'],
  ['#f472b6', '#fb923c'],
  ['#22d3ee', '#6366f1'],
  ['#34d399', '#3b82f6'],
  ['#f59e0b', '#ef4444'],
  ['#8b5cf6', '#ec4899'],
]
function fallbackStyle(i: number) {
  const [a, b] = GRADIENTS[i % GRADIENTS.length]
  return { '--grad-a': a, '--grad-b': b } as Record<string, string>
}

// ── Scroll reveal ─────────────────────────────────────────────
const sectionRef = ref<HTMLElement | null>(null)
const isVisible  = ref(false)
let observer: IntersectionObserver | null = null

// ── Carousel state ────────────────────────────────────────────
const carouselOverflow = ref<HTMLElement | null>(null)
const firstCardEl      = ref<HTMLElement | null>(null)
const trackOffset      = ref(0)
const currentIndex     = ref(0)

// Measured at runtime instead of hardcoded — this is what was breaking
// mobile: a fixed 360px width was used to compute the slide offset even
// though cards render at a different, fluid width on small screens.
const CARD_GAP = 24
const cardStep = ref(360 + CARD_GAP)

function setFirstCardRef(el: any) {
  firstCardEl.value = el
}

function measureCard() {
  if (!firstCardEl.value) return
  const rect = firstCardEl.value.getBoundingClientRect()
  const styles = getComputedStyle(firstCardEl.value.parentElement as Element)
  const gap = parseFloat(styles.gap || String(CARD_GAP)) || CARD_GAP
  cardStep.value = rect.width + gap
}

function getVisibleCount(): number {
  const w = carouselOverflow.value?.clientWidth ?? window.innerWidth
  if (w < 640)  return 1
  if (w < 1024) return 2
  return 3
}

const visibleCount = ref(3)

const maxIndex = computed(() =>
    Math.max(0, projects.value.length - visibleCount.value)
)

const atStart = computed(() => currentIndex.value <= 0)
const atEnd   = computed(() => currentIndex.value >= maxIndex.value)

const dotCount = computed(() => Math.ceil(projects.value.length / visibleCount.value))
const currentDot = computed(() => Math.floor(currentIndex.value / visibleCount.value))

function slide(dir: -1 | 1) {
  currentIndex.value = Math.max(0, Math.min(maxIndex.value, currentIndex.value + dir))
  updateOffset()
}

function goToDot(dotIndex: number) {
  currentIndex.value = Math.min(maxIndex.value, dotIndex * visibleCount.value)
  updateOffset()
}

function updateOffset() {
  trackOffset.value = -(currentIndex.value * cardStep.value)
}

async function handleResize() {
  visibleCount.value = getVisibleCount()
  await nextTick()
  measureCard()
  if (currentIndex.value > maxIndex.value) {
    currentIndex.value = maxIndex.value
  }
  updateOffset()
}

// ── Touch / swipe support (mobile) ───────────────────────────
const isDragging = ref(false)
const dragDelta   = ref(0)
let touchStartX = 0
let touchDeltaX = 0

function onTouchStart(e: TouchEvent) {
  if (projects.value.length <= visibleCount.value) return
  isDragging.value = true
  touchStartX = e.touches[0].clientX
  touchDeltaX = 0
}
function onTouchMove(e: TouchEvent) {
  if (!isDragging.value) return
  touchDeltaX = e.touches[0].clientX - touchStartX
  dragDelta.value = touchDeltaX
}
function onTouchEnd() {
  if (!isDragging.value) return
  isDragging.value = false
  const threshold = cardStep.value * 0.18
  if (touchDeltaX < -threshold) {
    slide(1)
  } else if (touchDeltaX > threshold) {
    slide(-1)
  }
  dragDelta.value = 0
  touchDeltaX = 0
}

// ── Lifecycle ─────────────────────────────────────────────────
onMounted(async () => {
  visibleCount.value = getVisibleCount()
  window.addEventListener('resize', handleResize)

  // Scroll reveal
  if (sectionRef.value && 'IntersectionObserver' in window) {
    observer = new IntersectionObserver(
        ([entry]) => {
          if (!entry.isIntersecting) return
          isVisible.value = true
          observer?.disconnect()
          observer = null
        },
        { rootMargin: '0px 0px -15% 0px', threshold: 0.1 }
    )
    observer.observe(sectionRef.value)
  } else {
    isVisible.value = true
  }

  // Fetch
  const { data, error } = await supabase
      .from('projects')
      .select('id, title, description, cover_image_url, tech_stack, live_url, github_url, featured, order_index')
      .eq('published', true)
      .order('order_index', { ascending: true })
  if (!error) projects.value = data ?? []
  loading.value = false

  await nextTick()
  measureCard()
  updateOffset()
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  observer?.disconnect()
})
</script>

<style scoped>
/* ── Section shell ────────────────────────────────────────── */
.projects-section {
  background: linear-gradient(180deg, #f8fafc 0%, #ffffff 100%);
  border-top: 1px solid rgba(108, 99, 255, 0.08);
  padding: 96px 0 112px;
  overflow: hidden;
  max-width: 100vw;
}

.section-inner {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 40px;
  box-sizing: border-box;
}

/* ── Reveal animation ─────────────────────────────────────── */
.reveal-item {
  opacity: 0;
  transform: translateY(28px);
  transition: opacity 0.75s cubic-bezier(0.16, 1, 0.3, 1),
  transform 0.75s cubic-bezier(0.16, 1, 0.3, 1);
  will-change: opacity, transform;
}

.projects-section.is-visible .reveal-item {
  opacity: 1;
  transform: translateY(0);
}

.projects-section.is-visible .section-header { transition-delay: 0.05s; }
.projects-section.is-visible .carousel-wrapper { transition-delay: 0.15s; }

/* ── Section header ───────────────────────────────────────── */
.section-header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 24px;
  margin-bottom: 48px;
  flex-wrap: wrap;
}

.section-kicker {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 700;
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.08);
  border: 1px solid rgba(108, 99, 255, 0.2);
  padding: 5px 14px;
  border-radius: 100px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  margin-bottom: 14px;
}

.header-left h2 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(28px, 3.5vw, 44px);
  font-weight: 700;
  color: #1a1a2e;
  line-height: 1.1;
  letter-spacing: -1px;
  margin: 0 0 10px;
}

.section-desc {
  font-family: 'Inter', sans-serif;
  font-size: clamp(14px, 1.2vw, 16px);
  color: #64748b;
  line-height: 1.7;
  margin: 0;
  max-width: 520px;
}

.view-all-btn {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: #6c63ff;
  text-decoration: none;
  padding: 10px 20px;
  border: 1.5px solid rgba(108, 99, 255, 0.3);
  border-radius: 12px;
  background: rgba(108, 99, 255, 0.05);
  transition: all 0.22s;
  white-space: nowrap;
  flex-shrink: 0;
}
.view-all-btn:hover {
  background: rgba(108, 99, 255, 0.11);
  border-color: #6c63ff;
  box-shadow: 0 4px 16px rgba(108, 99, 255, 0.14);
  transform: translateY(-1px);
}

/* ── Carousel ─────────────────────────────────────────────── */
.carousel-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  gap: 0;
  min-width: 0; /* prevents flex children from forcing overflow */
}

.carousel-overflow {
  flex: 1;
  min-width: 0;
  overflow: hidden;
  margin: 0 -4px;
  padding: 8px 4px 16px;
  /* soft fade at the edges so the peeking card feels intentional, not clipped */
  -webkit-mask-image: linear-gradient(90deg, transparent 0, #000 24px, #000 calc(100% - 24px), transparent 100%);
  mask-image: linear-gradient(90deg, transparent 0, #000 24px, #000 calc(100% - 24px), transparent 100%);
}

.carousel-track {
  display: flex;
  gap: 24px;
  transition: transform 0.45s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: transform;
}
.carousel-track.no-transition { transition: none; }

/* ── Carousel navigation buttons ─────────────────────────── */
.carousel-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1.5px solid rgba(108, 99, 255, 0.2);
  background: #fff;
  color: #6c63ff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  flex-shrink: 0;
  transition: all 0.2s;
  box-shadow: 0 2px 12px rgba(108, 99, 255, 0.08);
  z-index: 2;
}
.carousel-btn:hover:not(:disabled) {
  background: #6c63ff;
  color: #fff;
  border-color: #6c63ff;
  box-shadow: 0 4px 20px rgba(108, 99, 255, 0.25);
  transform: scale(1.06);
}
.carousel-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
  transform: none;
}
.carousel-btn.prev { margin-right: 14px; }
.carousel-btn.next { margin-left: 14px; }

/* ── Dot indicators ───────────────────────────────────────── */
.carousel-dots {
  display: flex;
  justify-content: center;
  gap: 8px;
  margin-top: 28px;
}
.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  border: none;
  background: rgba(108, 99, 255, 0.2);
  cursor: pointer;
  transition: all 0.25s;
  padding: 0;
}
.dot.active {
  background: #6c63ff;
  width: 24px;
  border-radius: 4px;
}

/* ── Project card ─────────────────────────────────────────── */
.project-card {
  width: 360px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  background: #fff;
  border: 1px solid rgba(108, 99, 255, 0.1);
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(15, 23, 42, 0.05);
  position: relative;
  transition: transform 0.28s ease, box-shadow 0.28s ease, border-color 0.28s ease;
}
.project-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 20px 56px rgba(108, 99, 255, 0.16);
  border-color: rgba(108, 99, 255, 0.28);
}
.project-card.featured {
  border-color: rgba(108, 99, 255, 0.28);
  background: linear-gradient(180deg, rgba(108,99,255,0.025) 0%, #fff 50%);
  box-shadow: 0 6px 24px rgba(108, 99, 255, 0.1);
}

.featured-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: #92400e;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(245,158,11,0.35);
  padding: 4px 10px 4px 8px;
  border-radius: 999px;
  z-index: 2;
  box-shadow: 0 2px 8px rgba(245, 158, 11, 0.2);
}
.featured-badge svg { color: #f59e0b; }

/* ── Cover / media ────────────────────────────────────────── */
.card-cover {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  overflow: hidden;
  background: #0f0f23;
  flex-shrink: 0;
}
.card-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s cubic-bezier(0.16, 1, 0.3, 1);
  display: block;
}
.project-card:hover .card-cover img { transform: scale(1.08); }

/* animated gradient-mesh placeholder used when there is no cover image */
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
  transition: transform 0.6s ease;
}
.cover-fallback .blob-a { width: 140px; height: 140px; background: #fff; top: -30px; left: -20px; }
.cover-fallback .blob-b { width: 110px; height: 110px; background: #000; bottom: -30px; right: -10px; }
.cover-fallback .blob-c { width: 90px;  height: 90px;  background: #fff; bottom: 10px; left: 40%; }
.project-card:hover .cover-fallback .blob-a { transform: translate(10px, 10px) scale(1.1); }
.project-card:hover .cover-fallback .blob-b { transform: translate(-8px, -8px) scale(1.1); }
.fallback-icon {
  position: relative;
  z-index: 1;
  color: rgba(255, 255, 255, 0.85);
}

/* subtle bottom scrim on every cover for consistent polish */
.cover-scrim {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(0,0,0,0) 55%, rgba(15,15,35,0.28) 100%);
  pointer-events: none;
}

/* quick-action overlay revealed on hover / tap */
.cover-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: rgba(15, 15, 35, 0.0);
  opacity: 0;
  transition: opacity 0.25s ease, background 0.25s ease;
}
.project-card:hover .cover-overlay {
  opacity: 1;
  background: rgba(15, 15, 35, 0.32);
}
.overlay-btn {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.95);
  color: #1a1a2e;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translateY(8px);
  opacity: 0;
  transition: transform 0.25s ease, opacity 0.25s ease, background 0.2s ease;
  box-shadow: 0 4px 14px rgba(0,0,0,0.25);
}
.project-card:hover .overlay-btn { transform: translateY(0); opacity: 1; }
.overlay-btn:hover { background: #6c63ff; color: #fff; }

/* Body */
.card-body {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px 22px 22px;
  flex: 1;
}

.card-title {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 17px;
  font-weight: 700;
  color: #1a1a2e;
  margin: 0;
  line-height: 1.3;
  letter-spacing: -0.2px;
}

.card-desc {
  font-family: 'Inter', sans-serif;
  font-size: 13.5px;
  line-height: 1.7;
  color: #64748b;
  margin: 0;
  flex: 1;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Tech */
.tech-stack {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}
.tech-badge {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: #312e81;
  background: rgba(108, 99, 255, 0.07);
  border: 1px solid rgba(108, 99, 255, 0.14);
  padding: 3px 9px;
  border-radius: 999px;
}

/* Footer links */
.card-footer {
  display: flex;
  gap: 7px;
  margin-top: 4px;
  flex-wrap: wrap;
}
.card-link {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-family: 'Inter', sans-serif;
  font-size: 12.5px;
  font-weight: 600;
  text-decoration: none;
  padding: 5px 12px;
  border-radius: 9px;
  border: 1.5px solid;
  transition: all 0.18s;
}
.card-link.live {
  color: #6c63ff;
  border-color: rgba(108,99,255,0.22);
  background: rgba(108,99,255,0.05);
}
.card-link.live:hover { background: rgba(108,99,255,0.12); border-color: #6c63ff; }
.card-link.github {
  color: #1a1a2e;
  border-color: rgba(26,26,46,0.14);
  background: rgba(26,26,46,0.03);
}
.card-link.github:hover { background: rgba(26,26,46,0.08); border-color: rgba(26,26,46,0.28); }
.card-link.detail {
  color: #6c63ff;
  border-color: rgba(108,99,255,0.22);
  background: rgba(108,99,255,0.05);
  margin-left: auto;
}
.card-link.detail:hover { background: rgba(108,99,255,0.12); border-color: #6c63ff; }

/* ── Skeleton ─────────────────────────────────────────────── */
.skeleton-card { pointer-events: none; }
.skeleton-img {
  width: 100%;
  aspect-ratio: 16 / 9;
  background: linear-gradient(90deg, #f1f5f9 25%, #e2e8f0 50%, #f1f5f9 75%);
  background-size: 400% 100%;
  animation: shimmer 1.5s ease infinite;
}
.skeleton-body {
  padding: 20px 22px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.skeleton-line {
  height: 13px;
  border-radius: 6px;
  background: linear-gradient(90deg, #f1f5f9 25%, #e2e8f0 50%, #f1f5f9 75%);
  background-size: 400% 100%;
  animation: shimmer 1.5s ease infinite;
}
.skeleton-line.w70 { width: 70%; }
.skeleton-line.w50 { width: 50%; }
.skeleton-line.w30 { width: 30%; }
@keyframes shimmer {
  0%   { background-position: 100% 0; }
  100% { background-position: -100% 0; }
}

/* ── Empty ────────────────────────────────────────────────── */
.empty-state {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 48px 0;
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
}
.empty-state span { font-size: 1.5rem; }

/* ── Responsive ───────────────────────────────────────────── */
@media (max-width: 1100px) {
  .project-card { width: 320px; }
}

@media (max-width: 900px) {
  .projects-section { padding: 72px 0 88px; }
  .section-inner { padding: 0 24px; }
  .section-header { flex-direction: column; align-items: flex-start; gap: 16px; }
  .project-card { width: 300px; }
}

/* Mobile: single-card "peek" carousel + swipe. Arrows move off to the
   sides of the content column instead of squeezing the card, and the
   card width is a % of the available track so cardStep (measured live
   in JS) always matches what's on screen — no more clipped edges. */
@media (max-width: 640px) {
  .projects-section { padding: 56px 0 64px; }
  .section-inner { padding: 0 16px; }
  .section-header { margin-bottom: 28px; }

  .carousel-wrapper { gap: 6px; }
  .carousel-btn { width: 34px; height: 34px; flex-shrink: 0; }
  .carousel-btn.prev { margin-right: 0; }
  .carousel-btn.next { margin-left: 0; }
  .carousel-btn svg { width: 16px; height: 16px; }

  .carousel-overflow {
    -webkit-mask-image: linear-gradient(90deg, transparent 0, #000 10px, #000 calc(100% - 10px), transparent 100%);
    mask-image: linear-gradient(90deg, transparent 0, #000 10px, #000 calc(100% - 10px), transparent 100%);
  }

  .project-card { width: 82vw; max-width: 340px; touch-action: pan-y; }
  .card-body { padding: 16px 18px 18px; }
  .card-title { font-size: 16px; }
}

@media (prefers-reduced-motion: reduce) {
  .reveal-item { opacity: 1; transform: none; transition: none; }
  .carousel-track { transition: none; }
}
</style>