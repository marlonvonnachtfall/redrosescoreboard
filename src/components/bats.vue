<script setup>
import { ref } from "vue";

const N = 12;
const directions = ["right", "left", "down", "up", "diag1", "diag2"];
const bats = ref(
  Array.from({ length: N }, () => {
    // Choose random direction
    const dir = directions[Math.floor(Math.random() * directions.length)];
    // Random starting position offset
    const top = 10 + Math.random() * 75; // vh
    const left = 10 + Math.random() * 80; // vw
    const scale = 0.6 + Math.random() * 0.8;
    const duration = 12 + Math.random() * 10;
    const delay = Math.random() * 8;
    return { dir, top, left, scale, duration, delay };
  })
);
</script>

<template>
  <div class="bats">
    <figure
      v-for="(b, i) in bats"
      :key="i"
      class="bat"
      :class="b.dir"
      :style="{
        '--top': b.top + 'vh',
        '--left': b.left + 'vw',
        '--dur': b.duration + 's',
        '--delay': b.delay + 's',
        '--scale': b.scale
      }"
    >
      <img
        src="/bat.gif"
        alt="Bat"
        class="bat-gif"
        :style="{ transform: `scale(${b.scale})` }"
      />
    </figure>
  </div>
</template>

<style scoped>
.bats {
  position: fixed; inset: 0;
  pointer-events: none;
  overflow: hidden;
  z-index: 5;
}
.bat {
  position: absolute;
  opacity: 0.9;
  filter: drop-shadow(0 1px 2px rgba(0,0,0,0.5));
}
.bat-gif {
  width: 64px; height: 32px;
  pointer-events: none;
}

/* Fly left to right */
.bat.right {
  top: var(--top);
  left: -10vw;
  animation: fly-right var(--dur) linear var(--delay) infinite;
}
@keyframes fly-right {
  0%   { transform: translateX(0); }
  100% { transform: translateX(110vw);}
}

/* Fly right to left */
.bat.left {
  top: var(--top);
  left: 110vw;
  animation: fly-left var(--dur) linear var(--delay) infinite;
}
@keyframes fly-left {
  0%   { transform: translateX(0) scaleX(-1);}
  100% { transform: translateX(-110vw) scaleX(-1);}
}

/* Fly top to bottom */
.bat.down {
  top: -10vh;
  left: var(--left);
  animation: fly-down var(--dur) linear var(--delay) infinite;
}
@keyframes fly-down {
  0%   { transform: translateY(0) rotate(90deg);}
  100% { transform: translateY(110vh) rotate(90deg);}
}

/* Fly bottom to top */
.bat.up {
  top: 110vh;
  left: var(--left);
  animation: fly-up var(--dur) linear var(--delay) infinite;
}
@keyframes fly-up {
  0%   { transform: translateY(0) rotate(-90deg);}
  100% { transform: translateY(-110vh) rotate(-90deg);}
}

/* Diagonal top-left to bottom-right */
.bat.diag1 {
  top: -10vh;
  left: -10vw;
  animation: fly-diag1 var(--dur) linear var(--delay) infinite;
}
@keyframes fly-diag1 {
  0%   { transform: translate(0,0) rotate(25deg);}
  100% { transform: translate(110vw,110vh) rotate(25deg);}
}

/* Diagonal bottom-right to top-left */
.bat.diag2 {
  top: 110vh;
  left: 110vw;
  animation: fly-diag2 var(--dur) linear var(--delay) infinite;
}
@keyframes fly-diag2 {
  0%   { transform: translate(0,0) rotate(-25deg) scaleX(-1);}
  100% { transform: translate(-110vw,-110vh) rotate(-25deg) scaleX(-1);}
}
</style>
