<template>
  <div class="guild-container">
    <Bats />
    <HeartExplode />

  <audio ref="entryAudio" src="/marathon-theme.mp3" preload="auto"></audio>
  <div
    v-if="audioBlocked"
    class="audio-gate"
    @click="resumeAudio"
    @touchstart.prevent="resumeAudio"
  >
    <p>🔊 Tap or click to enable sound</p>
  </div>
    <div class="guild-image"></div>

    <h1 class="guild-title">Red Rose's</h1>
    <h2 class="guild-title">Crimson Weekend</h2>
    <div class="search-bar">
      <input
        type="text"
        placeholder="Search for Roses..."
        v-model="searchQuery"
        @input="delaySearch"
      />

    </div>

    <div class="coinContainer">
      <div class="coinBackgroundLeft">
        <img src="/crystalcoins.gif" />
      </div>
      <div class="coinBackgroundRight">
        <img src="/crystalcoins.gif" />
      </div>
    </div>
    <div v-if="filteredScores.length" class="score-list">
      <div
        v-for="(score, index) in sortedFilteredScores"
        :key="score.name"
        :class="['score-item', crownBorder[index]]"
        :style="index < 3 ? 'margin-bottom:20px;' : ''"
      >
        <div class="crownContainer">
          <div
            v-if="index < 3"
            :style="{ backgroundImage: 'url(' + crowns[index] + '.png' + ')' }"
            class="crownLeft"
          ></div>
          <div
            v-if="index < 3"
            :style="{ backgroundImage: 'url(' + crowns[index] + '.png' + ')' }"
            class="crownRight"
          ></div>
        </div>
        <div class="character-card">
          <div class="flexColumn">
            <div class="flexRow">
              <div class="placement">#{{ score.placement }}</div>
              <div class="character-name">{{ score.name }}</div>
            </div>
            <div class="level">Level at start: {{ score.initialLevel }}</div>
          </div>
          <div class="flexColumn">
            <div class="character-points">Points: {{ score.points }}</div>
            <div class="level">Level Now: {{ score.levelNow }}</div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="loading">
      <p>Loading data or no members found...</p>
    </div>
  </div>
  <main class="spooky-bg test-border" style="min-height:100%;">
    <h1></h1>
  </main>
</template>

<style>
/* keep this unscoped or empty; global rules live in style.css */
</style>

<script>
import Bats from "./components/bats.vue";
import HeartExplode from "./components/heart-explode.vue";
export default {
  components: { Bats, HeartExplode },
  data() {
    return {
      searchQuery: "",
      guildMembers: [],
      initialLevels: [],
      scores: [],
      filteredScores: [],
      error: null,
      searchTimeout: null,
      crowns: ["goldCrown", "silverCrown", "bronzeCrown"],
      crownBorder: ["gold", "silver", "bronze"],
      audioBlocked: false, // overlay flag
    };
  },

  computed: {
    sortedFilteredScores() {
      const filtered = this.filteredScores.filter((s) => s.points > 0);
      return filtered
        .sort((a, b) => b.points - a.points)
        .map((score, index) => ({ ...score, placement: index + 1 }));
    },
  },

  methods: {
    delaySearch() {
      if (this.searchTimeout) clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(() => this.performSearch(), 560);
    },

    performSearch() {
      let filtered = this.scores.filter((s) => s.points > 0);
      if (this.searchQuery) {
        const q = this.searchQuery.toLowerCase();
        filtered = filtered.filter((s) => s.name.toLowerCase().includes(q));
      }
      this.filteredScores = filtered;
    },

    async fetchGuildData() {
      try {
        const response = await fetch("https://api.tibiadata.com/v4/guild/above");
        if (!response.ok) throw new Error("Failed to fetch guild data");
        const data = await response.json();
        if (data.guild && data.guild.members) {
          this.guildMembers = data.guild.members.map((m) => ({
            name: m.name,
            level: m.level,
          }));
        }
      } catch (err) {
        this.error = err.message;
        console.error("Error fetching guild data:", err);
      }
    },

    async fetchInitialLevels() {
      try {
        const response = await fetch("initial-level.json");
        if (!response.ok) throw new Error("Failed to load initial levels");
        this.initialLevels = await response.json();
      } catch (err) {
        this.error = err.message;
        console.error("Error fetching initial levels:", err);
      }
    },

    calculateScores() {
      if (this.guildMembers.length && this.initialLevels.length) {
        this.scores = this.guildMembers.map((member) => {
          const init = this.initialLevels.find((i) => i.name === member.name);
          if (init) {
            const diff = member.level - init.level;
            const points = this.calculatePoints(diff, init.level);
            return { name: member.name, points, initialLevel: init.level, levelNow: member.level };
          }
          return { name: member.name, points: 0, initialLevel: null, levelNow: member.level };
        });
        this.filteredScores = this.scores;
      }
    },

    calculatePoints(levelDifference, initialLevel) {
      let points = 0;
      for (let i = 1; i <= levelDifference; i++) {
        const level = initialLevel + i;
        if (level <= 49) points += 1;
        else if (level <= 124) points += 5;
        else if (level <= 224) points += 15;
        else if (level <= 349) points += 25;
        else if (level <= 499) points += 40;
        else if (level <= 674) points += 55;
        else if (level <= 875) points += 75;
        else points += 80;
      }
      return points;
    },

    async loadData() {
      await this.fetchInitialLevels();
      await this.fetchGuildData();
      this.calculateScores();
    },

    // ✅ audio methods live INSIDE methods {}
    async tryAutoplay() {
      const el = this.$refs.entryAudio;
      if (!el) return;
      try {
        el.volume = 0.8;
        // el.loop = true; // enable if you want looping
        await el.play();
        this.audioBlocked = false;
      } catch {
        this.audioBlocked = true; // show overlay if blocked by browser
      }
    },

    resumeAudio() {
      const el = this.$refs.entryAudio;
      if (!el) return;
      el.play();
      this.audioBlocked = false;
    },
  },

  created() {
    this.loadData();
  },

  mounted() {
    this.tryAutoplay();
  },
};
</script>
 

<style scoped>
/* Container Styles */
html {
  background-color: #242424;
  font-size: 12px;
}
.flexColumn {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  text-align: left;
}
.flexRow {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
.guild-container {
  padding: 45px;
  border-radius: 10px;
  max-width: 620px;
  margin: 0 auto;
  color: #fff;
  font-family: "Cinzel", serif;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
  border: 3px solid #c39b77;
  position: relative;
}

.guild-image {
  background-image: url("/above.png");
  background-size: contain;
  width: 100%;
  height: 150px;
  background-position: center;
  background-repeat: no-repeat;
}

.coinContainer {
  display: flex;
  justify-content: space-between;
  position: absolute;
  top: 310px;
  left: 10px;
  width: 100%;
  height: auto;
  pointer-events: none;
}
.coinBackgroundLeft {
  width: 80px;
  height: auto;
  image-rendering: pixelated;
}
.coinBackgroundRight {
  width: 80px;
  height: auto;
  image-rendering: pixelated;
  padding-right: 20px;
}
img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.guild-title {
  text-align: center;
  font-size: 2.5em;
  margin-bottom: 5px;
  color: #ffc107;
  text-shadow: 0 0 10px #000;
  margin-top: 0px;
}

.score-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.score-item {
  position: relative;
  background: rgba(0, 0, 0, 0.7);
  padding: 10px;
  /*   border-radius: 8px;
  border: 2px solid #c39b77; */
  transition: transform 0.3s ease;
}

.crownContainer {
  display: flex;
  justify-content: space-between;
  position: absolute;
  top: -35px;
  left: -50px;
  width:calc(100% + 105px);
  height: auto;
}
.crownLeft {
  background-size: contain;
  background-repeat: no-repeat;
  width: 112px;
  height: 60px;
  top: -38px;
  left: -50px;
  transform: rotate(-15deg);
  background-position: center;
}

.crownRight {
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
  width: 112px;
  height: 60px;
  top: -38px;
  left: 438px;
  transform: rotate(15deg);
}

.score-item:hover {
  transform: scale(1.05);
}

.character-card {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 10px;
  background: #4a3b2f;
  border-radius: 8px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
}

.character-name {
  font-size: 1.2em;
  font-weight: bold;
  color: #ffd700;
  text-align: left;
}

.placement {
  padding-right: 3px;
  font-size: 1.2em;
}

.character-points {
  font-size: 1.2em;
  color: #fff;
}

/* Search Bar Styles */
.search-bar {
  margin-bottom: 50px;
  margin-top: 20px;
  text-align: center;
}

.search-bar input {
  width: 66%;
  padding: 10px;
  font-size: 1.2em;
  border-radius: 8px;
  border: 2px solid #c39b77;
  background-color: #2f1e13;
  color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  transition: box-shadow 0.3s ease;
}

.search-bar input::placeholder {
  color: #c39b77;
}

.search-bar input:focus {
  outline: none;
  box-shadow: 0 0 15px #ffc107;
}

.loading {
  text-align: center;
  font-size: 1.2em;
  color: #fff;
}
/* Leader (Gold) */
.gold {
  animation: leader-glow 1.5s infinite alternate;
}

@keyframes leader-glow {
  0% {
    box-shadow: 0 0 5px #ffd700, 0 0 15px #ffd700, 0 0 20px #ffd700;
  }
  100% {
    box-shadow: 0 0 10px #ffc107, 0 0 15px #ffc107, 0 0 20px #ffc107;
  }
}

/* Second Place (Silver) */
.silver {
  animation: second-glow 1.5s infinite alternate;
}

@keyframes second-glow {
  0% {
    box-shadow: 0 0 5px #c0c0c0, 0 0 15px #c0c0c0, 0 0 20px #c0c0c0;
  }
  100% {
    box-shadow: 0 0 10px #b0b0b0, 0 0 15px #b0b0b0, 0 0 20px #b0b0b0;
  }
}

/* Third Place (Bronze) */
.bronze {
  animation: third-glow 1.5s infinite alternate;
}

@keyframes third-glow {
  0% {
    box-shadow: 0 0 5px #cd7f32, 0 0 15px #cd7f32, 0 0 20px #cd7f32;
  }
  100% {
    box-shadow: 0 0 10px #b87333, 0 0 15px #b87333, 0 0 20px #b87333;
  }
}

/* --- audio gate overlay --- */
.audio-gate{
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  cursor: pointer;
  text-align: center;
}
.audio-gate p{
  margin: 0;
  padding: 14px 18px;
  border: 1px solid rgba(255,255,255,.25);
  border-radius: 10px;
  background: rgba(20,20,26,.6);
  color: #fff;
  font-size: 1.05rem;
}

</style>
