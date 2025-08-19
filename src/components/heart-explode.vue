<script setup>
import { ref } from "vue";

const clickCount = ref(0);
const exploded = ref(false);

function handleClick() {
  if (exploded.value) return;
  clickCount.value++;
  if (clickCount.value >= 10) {
    exploded.value = true;
    // Play audio
    const audio = new Audio("/explosion.mp3");
    audio.play();
    setTimeout(() => {
      exploded.value = false;
      clickCount.value = 0;
    }, 2600); // Hide after GIF duration (adjust as needed)
  }
}
</script>

<template>
  <div class="heart-explode">
    <button
      class="heart-btn"
      :disabled="exploded"
      @click="handleClick"
      :aria-label="'Click me! (' + clickCount + '/10)'"
    >
      <span
        class="heart"
        :class="{ pulse: !exploded && clickCount > 0, shake: clickCount >= 9 }"
      >💖</span>
      <span v-if="!exploded" class="count">{{ 10 - clickCount }}</span>
    </button>
    <div v-if="exploded" class="explosion-overlay">
      <img src="/explosion.gif" alt="Explosion!" class="explosion-gif" />
      <div class="boom-text">BOOM!</div>
    </div>
  </div>
</template>

<style scoped>
.heart-explode {
  position: absolute;
  top: 18px;
  right: -82px;
  z-index: 20;
}

.heart-btn {
  background: #272727;
  border: 2px solid #c39b77;
  border-radius: 50%;
  width: 56px; height: 56px;
  font-size: 2.1em;
  cursor: pointer;
  box-shadow: 0 3px 18px #222a;
  position: relative;
  transition: border-color 0.25s;
}

.heart {
  display: inline-block;
  transition: transform 0.2s;
}
.heart.pulse {
  animation: pulse 0.22s;
}
.heart.shake {
  animation: shake 0.3s;
}
.count {
  position: absolute;
  top: 3px; right: 8px;
  font-size: 1em;
  color: #ffc107;
  font-weight: bold;
}

@keyframes pulse {
  0% { transform: scale(1); }
  70% { transform: scale(1.28); }
  100% { transform: scale(1); }
}
@keyframes shake {
  0% { transform: rotate(-10deg) scale(1.22); }
  40% { transform: rotate(10deg) scale(1.32); }
  70% { transform: rotate(-10deg) scale(1.22);}
  100% { transform: rotate(0deg) scale(1);}
}

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