<script setup>
import { ref, onMounted, onUnmounted } from "vue";

/* Number of floating lights/ghosts */
const N = 6;
const ghosts = ref(
  Array.from({ length: N }, (_, i) => ({
    x: Math.random() * 100,
    y: Math.random() * 60 + 10,
    size: 0.8 + Math.random() * 0.7,
    speed: 0.25 + Math.random() * 0.7,
    phase: Math.random() * Math.PI * 2,
    color: i % 2 ? "#e8e8ff" : "#b6e5f8"
  }))
);

const mouse = ref({ x: 50, y: 50 });
const width = ref(window.innerWidth);
const height = ref(window.innerHeight);

function handleMouse(e) {
  mouse.value.x = (e.clientX / width.value) * 100;
  mouse.value.y = (e.clientY / height.value) * 100;
}

let frame;
function animate() {
  ghosts.value.forEach((g, i) => {
    // Gentle float/wave
    g.phase += g.speed * 0.015;
    g.x += Math.sin(g.phase) * 0.08;
    g.y += Math.cos(g.phase + i) * 0.07;

    // Mouse attraction (subtle)
    g.x += (mouse.value.x - g.x) * 0.002;
    g.y += (mouse.value.y * 0.5 - g.y) * 0.001;
  });
  frame = requestAnimationFrame(animate);
}

onMounted(() => {
  window.addEventListener("mousemove", handleMouse);
  animate();
});
onUnmounted(() => {
  window.removeEventListener("mousemove", handleMouse);
  cancelAnimationFrame(frame);
});
</script>

<template>
  <div class="ghostlights">
    <figure
      v-for="(g, i) in ghosts"
      :key="i"
      class="ghost"
      :style="{
        left: g.x + 'vw',
        top: g.y + 'vh',
        transform: `scale(${g.size})`,
        opacity: 0.90 - i * 0.07,
        zIndex: 6 + i
      }"
    >
      <svg width="42" height="64" viewBox="0 0 42 64" aria-hidden="true">
        <radialGradient id="glow" cx="50%" cy="40%" r="60%">
          <stop offset="0%" :stop-color="g.color" stop-opacity="0.94"/>
          <stop offset="90%" :stop-color="g.color" stop-opacity="0"/>
        </radialGradient>
        <ellipse cx="21" cy="32" rx="16" ry="23" :fill="'url(#glow)'"/>
        <!-- Faint ghost face -->
        <ellipse cx="21" cy="38" rx="8" ry="14" fill="white" opacity="0.13"/>
        <ellipse cx="17" cy="38" rx="1.2" ry="2.5" fill="#222" opacity="0.5"/>
        <ellipse cx="25" cy="38" rx="1.2" ry="2.5" fill="#222" opacity="0.5"/>
        <ellipse cx="21" cy="44" rx="1.8" ry="0.7" fill="#222" opacity="0.32"/>
      </svg>
    </figure>
  </div>
</template>

<style scoped>
.ghostlights {
  position: fixed; inset: 0;
  pointer-events: none;
  z-index: 6;
}
.ghost {
  position: absolute;
  transition: opacity 0.3s;
  will-change: left, top, opacity, transform;
  filter: blur(0.5px) drop-shadow(0 2px 8px #b6e5f8);
  animation: ghostfade 8s linear infinite alternate;
}
@keyframes ghostfade {
  0%   { opacity: 0.96; }
  90%  { opacity: 0.77; }
  100% { opacity: 0.7; }
}
</style>