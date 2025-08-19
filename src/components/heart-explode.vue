<script setup>
import { ref } from "vue";

const exploded = ref(false);

// Preload the audio once. File should be in /public/explosion.mp3
const audio = new Audio("/explosion.mp3");
audio.preload = "auto";

async function handleClick() {
  if (exploded.value) return;         // ignore extra clicks while active
  exploded.value = true;
  try {
    audio.currentTime = 0;            // restart if it was played before
    await audio.play();
  } catch (_) {
    // If playback fails, the overlay still shows for 10s
  }

  // Keep the GIF visible for 10 seconds total
  setTimeout(() => {
    exploded.value = false;
  }, 10000);
}
</script>

<template>
  <div class="boom">
    <button
      class="boom-btn"
      :disabled="exploded"
      @click="handleClick"
      aria-label="Trigger effect"
      title="Trigger effect"
    >
      <span class="icon">📜</span>
    </button>

    <div v-if="exploded" class="explosion-overlay">
      <img src="/explosion.gif" alt="" class="explosion-gif" />
      <div class="boom-text">Praised be Banor!</div>
    </div>
  </div>
</template>

<style scoped>
.boom {
  position: absolute;
  top: 18px;
  right: -82px;
  z-index: 20;
}

.boom-btn {
  background: #272727;
  border: 2px solid #c39b77;
  border-radius: 50%;
  width: 56px; height: 56px;
  font-size: 2.1em;
  cursor: pointer;
  box-shadow: 0 3px 18px #222a;
  display: grid;
  place-items: center;
  transition: transform 0.08s ease, border-color 0.25s;
}
.boom-btn:active { transform: scale(0.96); }
.boom-btn:disabled { cursor: default; opacity: 0.85; }

.icon {
  line-height: 1;
  display: inline-block;
}

/* Overlay + GIF */
.explosion-overlay {
  position: fixed;
  inset: 0;
  background: rgba(32, 12, 18, 0.88);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 99;
}
.explosion-gif {
  width: 400px; max-width: 90vw;
  height: auto;
  margin-bottom: 16px;
}
.boom-text {
  font-size: 3em;
  color: #ffd700;
  text-shadow: 0 0 15px #b87333, 0 0 2px #fff;
  font-family: "Cinzel", serif;
  margin-top: 8px;
}
</style>
