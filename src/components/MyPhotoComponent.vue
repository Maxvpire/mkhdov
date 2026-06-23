<template>
  <canvas
    ref="canvasRef"
    @pointerdown="handlePointerMove"
    @pointermove="handlePointerMove"
    @pointerup="handlePointerEnd"
    @pointercancel="handlePointerEnd"
    @pointerleave="handlePointerEnd"
  />
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch, type PropType } from 'vue';
import portraitUrl from '../assets/mkhdov.png';

const props = defineProps({
  containerEl: {
    type: Object as PropType<HTMLElement | null>,
    default: null,
  },
});

const canvasRef = ref<HTMLCanvasElement | null>(null);

const GAP = 6;
const FONT_SIZE = 4;

const mouse = {
  x: 0,
  y: 0,
  active: false,
  radius: 80
};

let animationId = 0;
let particles: Particle[] = [];
let imgBounds = { x: 0, y: 0, width: 0, height: 0 };

function getContainerSize() {
  const el = props.containerEl ?? canvasRef.value?.parentElement ?? null;
  if (el) {
    return { w: el.clientWidth, h: el.clientHeight };
  }
  return { w: window.innerWidth, h: window.innerHeight };
}

class Particle {
  x: number;
  y: number;

  vx = 0;
  vy = 0;

  baseX: number;
  baseY: number;

  density: number;
  char: string;

  opacity = 0;
  delay = 0;
  elapsed = 0;

  constructor(
    x: number,
    y: number,
    brightness: number,
    delay: number
  ) {
    // Start scattered randomly around their base position
    this.x = x + (Math.random() - 0.5) * 800;
    this.y = y + (Math.random() - 0.5) * 800;

    this.baseX = x;
    this.baseY = y;

    this.density = Math.random() * 10 + 1;
    this.delay = delay;

    const chars = ['.', ':', '-', '+', '*', '#', '%', '@'];

    const idx = Math.floor(
      (brightness / 255) * (chars.length - 1)
    );

    this.char = chars[idx];
  }

  update() {
    this.elapsed += 1;
    if (this.elapsed < this.delay) return;

    if (this.opacity < 1) {
      this.opacity += 0.02;
    }

    // --- MOUSE INTERACTION ---
    if (mouse.active) {
      const dx = mouse.x - this.x;
      const dy = mouse.y - this.y;
      const distance = Math.sqrt(dx * dx + dy * dy);

      if (distance < mouse.radius) {
        const force = (mouse.radius - distance) / mouse.radius;
        const angle = Math.atan2(dy, dx);
        this.vx -= Math.cos(angle) * force * this.density * 0.4;
        this.vy -= Math.sin(angle) * force * this.density * 0.4;
      }
    }

    // --- ALWAYS APPLY SPRING FORCE ---
    // This ensures they return to their base position even without mouse input
    const spring = 0.04;
    this.vx += (this.baseX - this.x) * spring;
    this.vy += (this.baseY - this.y) * spring;

    this.vx *= 0.88;
    this.vy *= 0.88;

    this.x += this.vx;
    this.y += this.vy;
  }

  draw(ctx: CanvasRenderingContext2D) {
    if (this.elapsed < this.delay) return;
    ctx.globalAlpha = this.opacity;
    ctx.fillText(this.char, this.x, this.y);
    ctx.globalAlpha = 1.0;
  }
}

function handlePointerMove(e: PointerEvent) {
  const canvas = canvasRef.value!;
  const rect = canvas.getBoundingClientRect();
  mouse.x = e.clientX - rect.left;
  mouse.y = e.clientY - rect.top;
  mouse.active = true;

  if (e.pointerType !== 'mouse' && e.type === 'pointerdown') {
    canvas.setPointerCapture(e.pointerId);
  }

  if (
    mouse.x >= imgBounds.x - mouse.radius &&
    mouse.x <= imgBounds.x + imgBounds.width + mouse.radius &&
    mouse.y >= imgBounds.y - mouse.radius &&
    mouse.y <= imgBounds.y + imgBounds.height + mouse.radius
  ) {
    canvas.style.cursor = 'crosshair';
  } else {
    canvas.style.cursor = 'default';
  }
}

function handlePointerEnd(e?: PointerEvent) {
  if (e && canvasRef.value?.hasPointerCapture(e.pointerId)) {
    canvasRef.value.releasePointerCapture(e.pointerId);
  }

  mouse.active = false;
}

function resizeCanvas() {
  const canvas = canvasRef.value!;
  const dpr = window.devicePixelRatio || 1;
  const { w, h } = getContainerSize();

  canvas.width = w * dpr;
  canvas.height = h * dpr;

  canvas.style.width = `${w}px`;
  canvas.style.height = `${h}px`;

  const ctx = canvas.getContext('2d')!;

  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

  ctx.font = `${FONT_SIZE}px monospace`;
  ctx.fillStyle = '#6c63ff';
}

async function loadImage(url: string) {
  const img = new Image();

  return new Promise<HTMLImageElement>((resolve, reject) => {
    img.onload = () => resolve(img);
    img.onerror = (e) => reject(e);
    img.src = url;
  });
}

async function initParticles() {
  const img = await loadImage(portraitUrl);

  particles = [];

  const { w, h } = getContainerSize();

  const compactLayout = w <= 640 || h <= 360;

  const PORTRAIT_OFFSET_Y = -160;        // desktop — higher = move up
  const PORTRAIT_OFFSET_Y_COMPACT = -20; // mobile/tablet

  const scale = Math.min(
      (w * (compactLayout ? 0.78 : 0.9)) / img.width,
      (h * (compactLayout ? 0.86 : 0.9)) / img.height
  );

  const width  = img.width  * scale;
  const height = img.height * scale;

  const x = (w - width) / 2;
  const y = compactLayout
      ? (h - height) / 2 + PORTRAIT_OFFSET_Y_COMPACT
      : (h - height) / 2 + PORTRAIT_OFFSET_Y;

  imgBounds = { x, y, width, height };

  const offscreen = document.createElement('canvas');
  offscreen.width  = w;
  offscreen.height = h;
  const offCtx = offscreen.getContext('2d', { willReadFrequently: true })!;
  offCtx.drawImage(img, x, y, width, height);

  const imageData = offCtx.getImageData(0, 0, w, h);

  for (let py = 0; py < h; py += GAP) {
    for (let px = 0; px < w; px += GAP) {
      const index = (py * imageData.width + px) * 4;

      const r = imageData.data[index];
      const g = imageData.data[index + 1];
      const b = imageData.data[index + 2];
      const a = imageData.data[index + 3];

      if (a > 100 && (r < 240 || g < 240 || b < 240)) {
        const brightness = (r + g + b) / 3;
        const delay = (px + py) * 0.05;

        particles.push(
            new Particle(px, py, 255 - brightness, delay)
        );
      }
    }
  }
}

function animate() {
  const canvas = canvasRef.value!;
  const ctx = canvas.getContext('2d')!;
  const { w, h } = getContainerSize();

  ctx.clearRect(0, 0, w, h);

  for (const particle of particles) {
    particle.update();
    particle.draw(ctx);
  }

  animationId = requestAnimationFrame(animate);
}

async function reinit() {
  cancelAnimationFrame(animationId);
  resizeCanvas();
  try {
    await initParticles();
    animate();
  } catch (error) {
    console.error('Failed to load portrait or initialize particles:', error);
  }
}

onMounted(async () => {
  // Wait for the browser to finish its current paint cycle
  requestAnimationFrame(async () => {
    resizeCanvas();
    window.addEventListener('resize', reinit);
    try {
      await initParticles();
      animate();
    } catch (error) {
      console.error('Initialization failed:', error);
    }
  });
});

watch(() => props.containerEl, () => {
  reinit();
});

onUnmounted(() => {
  cancelAnimationFrame(animationId);
  window.removeEventListener('resize', reinit);
});
</script>

<style scoped>
canvas {
  display: block;
  background: transparent;
  touch-action: none;
  user-select: none;
}
</style>
