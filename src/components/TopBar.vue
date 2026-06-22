<template>
  <header class="topbar">
    <div class="topbar-inner">

      <!-- Left: Title + Nav -->
      <div class="topbar-left">
        <router-link to="/" class="topbar-title">mkhdov</router-link>
        <nav class="topbar-nav" id="main-nav">
          <a
            href="/"
            class="nav-link"
            :class="{ 'nav-link--active': isHomeActive }"
            :aria-current="isHomeActive ? 'page' : undefined"
            @click.prevent="goToHome"
          >
            Home
          </a>
          <a
            href="/#about"
            class="nav-link"
            :class="{ 'nav-link--active': isAboutActive }"
            :aria-current="isAboutActive ? 'page' : undefined"
            @click.prevent="goToAbout"
          >
            About
          </a>
          <a href="#" class="nav-link">Projects</a>
          <a href="#" class="nav-link">Contact</a>
        </nav>
      </div>

      <!-- Right: Socials + Articles -->
      <div class="topbar-right">
        <div class="topbar-socials">
          <!-- Gmail -->
          <a
            href="mailto:mkhdov@yahoo.com"
            class="social-icon"
            title="Email"
            aria-label="Send email"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/>
              <polyline points="22,6 12,13 2,6"/>
            </svg>
          </a>

          <!-- LinkedIn -->
          <a
            href="https://linkedin.com/in/mkhdov"
            target="_blank"
            rel="noopener noreferrer"
            class="social-icon"
            title="LinkedIn"
            aria-label="LinkedIn profile"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"/>
              <rect x="2" y="9" width="4" height="12"/>
              <circle cx="4" cy="4" r="2"/>
            </svg>
          </a>

          <!-- GitHub -->
          <a
            href="https://github.com/mkhdov"
            target="_blank"
            rel="noopener noreferrer"
            class="social-icon"
            title="GitHub"
            aria-label="GitHub profile"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
            </svg>
          </a>
        </div>

        <!-- Divider -->
        <div class="nav-divider" aria-hidden="true"></div>

        <!-- Blog — icon only -->
        <router-link
          to="/blog"
          class="blog-link"
          active-class="blog-link--active"
          title="Blog"
          aria-label="Blog"
          id="blog-nav-link"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M12 20h9"/>
            <path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/>
          </svg>
        </router-link>

        <!-- Articles — after blog -->
        <router-link to="/articles" class="articles-link" active-class="articles-link--active" id="articles-nav-link">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
            <polyline points="14 2 14 8 20 8"/>
            <line x1="16" y1="13" x2="8" y2="13"/>
            <line x1="16" y1="17" x2="8" y2="17"/>
          </svg>
          <span>Articles</span>
        </router-link>
      </div>

      <!-- Mobile hamburger -->
      <button class="hamburger" id="hamburger-btn" @click="menuOpen = !menuOpen" :aria-expanded="menuOpen" aria-label="Toggle menu">
        <span class="bar" :class="{ open: menuOpen }"></span>
        <span class="bar" :class="{ open: menuOpen }"></span>
        <span class="bar" :class="{ open: menuOpen }"></span>
      </button>
    </div>

    <!-- Mobile menu drawer -->
    <transition name="drawer">
      <nav v-if="menuOpen" class="mobile-menu" id="mobile-menu" @click="menuOpen = false">
        <a
          href="/"
          class="mobile-nav-link"
          :class="{ 'mobile-nav-link--active': isHomeActive }"
          :aria-current="isHomeActive ? 'page' : undefined"
          @click.prevent="goToHome"
        >
          Home
        </a>
        <a
          href="/#about"
          class="mobile-nav-link"
          :class="{ 'mobile-nav-link--active': isAboutActive }"
          :aria-current="isAboutActive ? 'page' : undefined"
          @click.prevent="goToAbout"
        >
          About
        </a>
        <a href="#" class="mobile-nav-link">Projects</a>
        <a href="#" class="mobile-nav-link">Contact</a>
        <router-link to="/blog" class="mobile-nav-link mobile-blog" active-class="mobile-nav-link--active">Blog</router-link>
        <router-link to="/articles" class="mobile-nav-link mobile-articles" active-class="mobile-nav-link--active">Articles</router-link>
        <div class="mobile-socials">
          <a href="mailto:mkhdov@yahoo.com" class="social-icon" title="Email">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/>
              <polyline points="22,6 12,13 2,6"/>
            </svg>
          </a>
          <a href="https://linkedin.com/in/mkhdov" target="_blank" rel="noopener noreferrer" class="social-icon" title="LinkedIn">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"/>
              <rect x="2" y="9" width="4" height="12"/>
              <circle cx="4" cy="4" r="2"/>
            </svg>
          </a>
          <a href="https://github.com/mkhdov" target="_blank" rel="noopener noreferrer" class="social-icon" title="GitHub">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
            </svg>
          </a>
        </div>
      </nav>
    </transition>
  </header>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const menuOpen = ref(false);
const route = useRoute();
const router = useRouter();
const activeSection = ref<'home' | 'about'>('home');

const isHomeRoute = computed(() => route.path === '/');
const isHomeActive = computed(() => isHomeRoute.value && activeSection.value === 'home');
const isAboutActive = computed(() => isHomeRoute.value && activeSection.value === 'about');

let ticking = false;

function updateActiveSection() {
  if (!isHomeRoute.value) {
    activeSection.value = 'home';
    return;
  }

  const about = document.getElementById('about');

  if (!about) {
    activeSection.value = 'home';
    return;
  }

  const topbarOffset = window.innerWidth <= 640 ? 56 : 64;
  const activationLine = topbarOffset + Math.min(window.innerHeight * 0.35, 220);

  activeSection.value = about.getBoundingClientRect().top <= activationLine ? 'about' : 'home';
}

function requestActiveUpdate() {
  if (ticking) return;

  ticking = true;
  window.requestAnimationFrame(() => {
    updateActiveSection();
    ticking = false;
  });
}

async function goToHome() {
  menuOpen.value = false;

  if (route.path !== '/') {
    await router.push('/');
  } else if (route.hash) {
    await router.replace('/');
  }

  await nextTick();
  window.scrollTo({ top: 0, behavior: 'smooth' });
  activeSection.value = 'home';
}

async function goToAbout() {
  menuOpen.value = false;

  if (route.path !== '/' || route.hash !== '#about') {
    await router.push({ path: '/', hash: '#about' });
  }

  await nextTick();
  document.getElementById('about')?.scrollIntoView({ behavior: 'smooth', block: 'start' });
  activeSection.value = 'about';
}

onMounted(() => {
  window.addEventListener('scroll', requestActiveUpdate, { passive: true });
  window.addEventListener('resize', requestActiveUpdate);
  window.addEventListener('hashchange', requestActiveUpdate);

  nextTick(requestActiveUpdate);
});

onUnmounted(() => {
  window.removeEventListener('scroll', requestActiveUpdate);
  window.removeEventListener('resize', requestActiveUpdate);
  window.removeEventListener('hashchange', requestActiveUpdate);
});

watch(
  () => route.fullPath,
  () => nextTick(requestActiveUpdate)
);
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Inter:wght@300;400;500;600&display=swap');

/* ─── Topbar shell ───────────────────────────── */
.topbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(108, 99, 255, 0.08);
  box-shadow: 0 1px 20px rgba(108, 99, 255, 0.08), 0 4px 32px rgba(0, 0, 0, 0.04);
  animation: topBarSlide 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
}

@keyframes topBarSlide {
  from { opacity: 0; transform: translateY(-100%); }
  to { opacity: 1; transform: translateY(0); }
}

.topbar-inner {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 32px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

/* ─── Left ───────────────────────────────────── */
.topbar-left {
  display: flex;
  align-items: center;
  gap: 28px;
  min-width: 0;
}

.topbar-title {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 22px;
  font-weight: 700;
  letter-spacing: -0.5px;
  text-decoration: none;
  background: linear-gradient(135deg, #6c63ff 0%, #a78bfa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  transition: opacity 0.2s ease;
  flex-shrink: 0;
}

.topbar-title:hover { opacity: 0.75; }

.topbar-nav {
  display: flex;
  align-items: center;
  gap: 2px;
}

.nav-link {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #64748b;
  text-decoration: none;
  padding: 6px 12px;
  border-radius: 8px;
  white-space: nowrap;
  transition: color 0.2s ease, background 0.2s ease;
}

.nav-link:hover       { color: #6c63ff; background: rgba(108, 99, 255, 0.06); }
.nav-link--active     { color: #6c63ff; background: rgba(108, 99, 255, 0.09); }

/* ─── Right ──────────────────────────────────── */
.topbar-right {
  display: flex;
  align-items: center;
  gap: 6px;
}

.topbar-socials {
  display: flex;
  align-items: center;
  gap: 2px;
}

.social-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 10px;
  color: #94a3b8;
  text-decoration: none;
  transition: color 0.2s ease, background 0.2s ease, transform 0.2s ease;
}

.social-icon:hover {
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.08);
  transform: translateY(-1px);
}

/* Thin vertical divider between socials and blog/articles */
.nav-divider {
  width: 1px;
  height: 22px;
  background: rgba(108, 99, 255, 0.15);
  margin: 0 8px;
  flex-shrink: 0;
}

/* Blog — icon-only button */
.blog-link {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 10px;
  color: #94a3b8;
  text-decoration: none;
  transition: color 0.2s ease, background 0.2s ease, transform 0.2s ease;
}

.blog-link:hover {
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.08);
  transform: translateY(-1px);
}

.blog-link--active {
  color: #6c63ff;
  background: rgba(108, 99, 255, 0.1);
  box-shadow: inset 0 0 0 1px rgba(108, 99, 255, 0.18);
}

/* Articles pill — rightmost item */
.articles-link {
  font-family: 'Inter', sans-serif;
  font-size: 13.5px;
  font-weight: 600;
  color: #6c63ff;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 15px;
  border-radius: 10px;
  border: 1.5px solid rgba(108, 99, 255, 0.25);
  background: rgba(108, 99, 255, 0.05);
  white-space: nowrap;
  transition: background 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease;
}

.articles-link:hover {
  background: rgba(108, 99, 255, 0.1);
  border-color: rgba(108, 99, 255, 0.4);
  box-shadow: 0 2px 12px rgba(108, 99, 255, 0.15);
}

.articles-link--active {
  background: rgba(108, 99, 255, 0.12);
  border-color: #6c63ff;
}

/* ─── Hamburger (mobile only) ────────────────── */
.hamburger {
  display: none;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
  width: 40px;
  height: 40px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  border-radius: 8px;
  transition: background 0.2s;
  flex-shrink: 0;
}

.hamburger:hover { background: rgba(108, 99, 255, 0.07); }

.bar {
  display: block;
  width: 100%;
  height: 2px;
  background: #64748b;
  border-radius: 2px;
  transition: transform 0.25s ease, opacity 0.25s ease;
}

.bar.open:nth-child(1) { transform: translateY(7px) rotate(45deg); }
.bar.open:nth-child(2) { opacity: 0; transform: scaleX(0); }
.bar.open:nth-child(3) { transform: translateY(-7px) rotate(-45deg); }

/* ─── Mobile drawer ──────────────────────────── */
.mobile-menu {
  position: absolute;
  top: 64px;
  left: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.98);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-bottom: 1px solid rgba(108, 99, 255, 0.1);
  padding: 12px 20px 20px;
  display: flex;
  flex-direction: column;
  gap: 2px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
}

.mobile-nav-link {
  font-family: 'Inter', sans-serif;
  font-size: 15px;
  font-weight: 500;
  color: #475569;
  text-decoration: none;
  padding: 11px 14px;
  border-radius: 10px;
  transition: color 0.2s, background 0.2s;
}

.mobile-nav-link:hover         { color: #6c63ff; background: rgba(108, 99, 255, 0.06); }
.mobile-nav-link--active       { color: #6c63ff; background: rgba(108, 99, 255, 0.08); }

.mobile-articles {
  color: #6c63ff;
  font-weight: 600;
  border: 1.5px solid rgba(108, 99, 255, 0.25);
  background: rgba(108, 99, 255, 0.05);
  margin-top: 6px;
  text-align: center;
}

.mobile-blog {
  color: #6c63ff;
  font-weight: 600;
  border: 1.5px solid rgba(108, 99, 255, 0.18);
  background: rgba(108, 99, 255, 0.04);
  margin-top: 6px;
  text-align: center;
}

.mobile-socials {
  display: flex;
  gap: 4px;
  padding: 8px 4px 0;
  margin-top: 4px;
  border-top: 1px solid rgba(108, 99, 255, 0.08);
}

/* ─── Drawer animation ───────────────────────── */
.drawer-enter-active,
.drawer-leave-active {
  transition: opacity 0.2s ease, transform 0.25s ease;
}
.drawer-enter-from,
.drawer-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ─── Tablet (≤ 900px): hide text nav, keep socials+articles ─── */
@media (max-width: 900px) {
  .topbar-nav .nav-link:not(:first-child) {
    display: none;
  }
}

/* ─── Mobile (≤ 640px): full hamburger mode ──── */
@media (max-width: 640px) {
  .topbar-inner {
    padding: 0 16px;
    height: 56px;
  }

  .topbar-nav    { display: none; }
  .topbar-right  { display: none; }
  .hamburger     { display: flex; }

  .mobile-menu {
    top: 56px;
  }
}
</style>
