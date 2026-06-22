<template>
  <main class="not-found">
    <div class="not-found-inner">
      <!-- Glitch number -->
      <div class="error-code" aria-hidden="true">
        <span class="digit">4</span>
        <span class="digit digit--accent">0</span>
        <span class="digit">4</span>
      </div>

      <div class="not-found-content">
        <h1>Page Not Found</h1>
        <p>
          Looks like this page went on a little adventure and got lost somewhere in the code.
        </p>

        <router-link to="/" class="home-btn" id="back-home-btn">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/>
            <polyline points="9 22 9 12 15 12 15 22"/>
          </svg>
          Back to Home
        </router-link>
      </div>

      <!-- Floating particles -->
      <div class="float-particle p1" aria-hidden="true">.</div>
      <div class="float-particle p2" aria-hidden="true">:</div>
      <div class="float-particle p3" aria-hidden="true">*</div>
      <div class="float-particle p4" aria-hidden="true">#</div>
      <div class="float-particle p5" aria-hidden="true">@</div>
      <div class="float-particle p6" aria-hidden="true">+</div>
    </div>
  </main>
</template>

<script setup lang="ts">
// 404 Not Found page
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Inter:wght@300;400;500;600&display=swap');

.not-found {
  min-height: 100vh;
  background: #ffffff;
  padding-top: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  box-sizing: border-box;
  padding-left: 20px;
  padding-right: 20px;
}

.not-found-inner {
  position: relative;
  text-align: center;
  padding: 48px 32px;
  max-width: 600px;
}

/* ─── Error Code ────────────────────────── */
.error-code {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  margin-bottom: 32px;
}

.digit {
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(80px, 15vw, 160px);
  font-weight: 700;
  color: #e2e8f0;
  letter-spacing: -8px;
  line-height: 1;
  text-shadow: none;
  transition: color 0.3s;
}

.digit--accent {
  color: transparent;
  -webkit-text-stroke: 4px #6c63ff;
  position: relative;
  animation: pulseAccent 3s ease-in-out infinite;
}

@keyframes pulseAccent {
  0%, 100% { -webkit-text-stroke-color: #6c63ff; }
  50% { -webkit-text-stroke-color: #a78bfa; }
}

/* ─── Content ────────────────────────── */
.not-found-content h1 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: clamp(22px, 4vw, 36px);
  font-weight: 700;
  color: #1a1a2e;
  letter-spacing: -1px;
  margin: 0 0 16px;
}

.not-found-content p {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  line-height: 1.7;
  color: #64748b;
  margin: 0 0 36px;
}

.home-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-family: 'Inter', sans-serif;
  font-size: 15px;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(135deg, #6c63ff 0%, #818cf8 100%);
  text-decoration: none;
  padding: 13px 28px;
  border-radius: 14px;
  box-shadow: 0 4px 24px rgba(108, 99, 255, 0.3);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.home-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 32px rgba(108, 99, 255, 0.4);
}

/* ─── Floating ASCII Particles ────────── */
.float-particle {
  position: absolute;
  font-family: monospace;
  font-size: 20px;
  color: #6c63ff;
  pointer-events: none;
  opacity: 0.15;
  animation: floatUp linear infinite;
  user-select: none;
}

.p1 { left: 10%; bottom: -20px; animation-duration: 6s; animation-delay: 0s; font-size: 24px; }
.p2 { left: 25%; bottom: -20px; animation-duration: 8s; animation-delay: 1.5s; font-size: 18px; }
.p3 { left: 45%; bottom: -20px; animation-duration: 7s; animation-delay: 0.8s; font-size: 28px; }
.p4 { left: 60%; bottom: -20px; animation-duration: 9s; animation-delay: 2s; font-size: 20px; }
.p5 { left: 75%; bottom: -20px; animation-duration: 6.5s; animation-delay: 0.3s; font-size: 22px; }
.p6 { left: 90%; bottom: -20px; animation-duration: 8.5s; animation-delay: 1s; font-size: 16px; }

@keyframes floatUp {
  from {
    transform: translateY(0) rotate(0deg);
    opacity: 0.15;
  }
  80% {
    opacity: 0.15;
  }
  to {
    transform: translateY(-600px) rotate(360deg);
    opacity: 0;
  }
}

/* ─── Tablet (641px – 900px) ─────────────────── */
@media (max-width: 900px) {
  .not-found {
    padding-top: 64px;
    align-items: center;
  }

  .not-found-inner {
    padding: 32px 24px;
  }

  .digit {
    letter-spacing: -4px;
  }

  .not-found-content h1 {
    font-size: clamp(20px, 5vw, 30px);
  }
}

/* ─── Mobile (≤ 640px) ───────────────────────── */
@media (max-width: 640px) {
  .not-found {
    padding-top: 56px; /* shorter topbar */
  }

  .not-found-inner {
    padding: 24px 16px;
    width: 100%;
  }

  .error-code {
    margin-bottom: 20px;
    gap: 0;
  }

  .digit {
    letter-spacing: -6px;
  }

  .digit--accent {
    -webkit-text-stroke-width: 3px;
  }

  .not-found-content h1 {
    font-size: 22px;
    margin-bottom: 12px;
  }

  .not-found-content p {
    font-size: 15px;
    margin-bottom: 28px;
  }

  .home-btn {
    width: 100%;
    justify-content: center;
    padding: 13px 20px;
  }

  /* Fewer floating particles on mobile */
  .p2, .p4, .p6 { display: none; }
}
</style>
