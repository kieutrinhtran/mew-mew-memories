<template>
  <main-screen
    v-if="statusMatch === 'default'"
    @onStart="onHandleBeforeStart($event)"
  />
  <interact-screen
    v-if="statusMatch === 'match'"
    :cardsContext="settings.cardsContext"
    @onFinish="onGetResult"
  />
  <!-- Phát nhạc khi đang chơi -->
  <audio ref="gameMusic" src="/sounds/gameMusic.mp3" loop></audio>
  <div v-if="statusMatch === 'match'" class="timer-display">
    🕒 {{ elapsedSeconds }}s
  </div>

  <!-- Hiển thị nút Back khi đang ở màn chơi -->
  <button
    v-if="statusMatch === 'match'"
    class="back-button"
    @click="onBackToMain"
  >
    ⬅ Back
  </button>

  <!-- BACK button -->
  <button
    v-if="statusMatch === 'match'"
    class="back-button"
    @click="showConfirmExit = true"
  >
    ⬅ Back
  </button>

  <!-- Back Confirm Popup -->
  <div v-if="showConfirmExit" class="popup-overlay">
    <div class="popup">
      <p class="popup-message">⚠ Exit now? Your progress will be lost!</p>
      <div class="popup-buttons">
        <button class="yes-btn" @click="confirmExit">Yes</button>
        <button class="no-btn" @click="showConfirmExit = false">No</button>
      </div>
    </div>
  </div>

  <result-screen
    v-if="statusMatch === 'result'"
    :timer="timer"
    :elapsedSeconds="elapsedSeconds"
    @onStartAgain="statusMatch = 'default'"
  />
  <!-- Hiện copyright nếu đang ở màn hình chính -->
  <p class="copyright" v-if="statusMatch === 'default'">
    © 2025 - This flip card game was created to support the presentation of
    Group 3 for the Web Application Development course at UEH
  </p>
</template>

<script>
import MainScreen from "./components/MainScreen.vue";
import InteractScreen from "./components/InteractScreen.vue";
import ResultScreen from "./components/ResultScreen.vue";

import { shuffled } from "./utils/array";

export default {
  name: "App",
  components: {
    MainScreen,
    InteractScreen,
    ResultScreen,
  },
  data() {
    return {
      settings: {
        totalOfBlocks: 0,
        cardsContext: [],
        startedAt: null,
      },
      timer: 0,
      statusMatch: "default",
      intervalId: null, // Thêm biến interval cho timer
      elapsedSeconds: 0, // Thời gian hiển thị cho timer
      fadeInterval: null,
      showConfirmExit: false,
    };
  },
  methods: {
    onHandleBeforeStart(configs) {
      this.settings.totalOfBlocks = configs.totalOfBlocks;

      const firstCards = Array.from(
        { length: this.settings.totalOfBlocks / 2 },
        (_, i) => i + 1
      );
      const secondCards = [...firstCards];
      const cards = [...firstCards, ...secondCards];

      this.settings.cardsContext = shuffled(shuffled(shuffled(cards)));
      this.settings.startedAt = new Date().getTime();

      this.elapsedSeconds = 0;

      // Phát nhạc chơi game
      const music = this.$refs.gameMusic;
      if (music) {
        music.currentTime = 0;
        music.volume = 0.3;
        music.play().catch(() => {});

        // Fade-in: tăng volume dần lên 0.4
        this.fadeInterval = setInterval(() => {
          if (music.volume < 0.4) {
            music.volume = Math.min(0.4, music.volume + 0.02);
          } else {
            clearInterval(this.fadeInterval);
          }
        }, 100); // mỗi 100ms tăng nhẹ
      }

      // Start timer
      this.intervalId = setInterval(() => {
        this.elapsedSeconds = Math.floor(
          (new Date().getTime() - this.settings.startedAt) / 1000
        );
      }, 1000);
      this.statusMatch = "match";
    },

    onBackToMain() {
      // Nếu có nhạc hoặc timer thì tắt:
      clearInterval(this.intervalId);
      this.intervalId = null;

      const music = this.$refs.gameMusic;
      if (music) {
        music.pause();
        music.currentTime = 0;
      }

      this.statusMatch = "default"; // Quay về màn chính
    },

    confirmExit() {
      clearInterval(this.intervalId);
      this.intervalId = null;

      const music = this.$refs.gameMusic;
      if (music) {
        music.pause();
        music.currentTime = 0;
      }

      this.statusMatch = "default";
      this.showConfirmExit = false;
    },

    onGetResult() {
      const now = new Date().getTime();
      this.timer = now - this.settings.startedAt;

      // Đồng bộ giây
      this.elapsedSeconds = Math.round(this.timer / 1000);

      // Dừng nhạc chơi game với fade-out
      const music = this.$refs.gameMusic;
      if (music) {
        clearInterval(this.fadeInterval); // nếu đang fade-in thì dừng lại

        this.fadeInterval = setInterval(() => {
          if (music.volume > 0) {
            music.volume = Math.max(0, music.volume - 0.02);
          } else {
            clearInterval(this.fadeInterval);
            music.pause();
            music.currentTime = 0;
          }
        }, 100);
      }

      clearInterval(this.intervalId);
      this.intervalId = null;
      this.statusMatch = "result";
    },
  },
};
</script>

<style lang="css" scoped>
.copyright {
  font-family: "Shantell Sans", cursive;
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  bottom: 1rem;
  color: var(--light);
  z-index: 3;
  font-size: 0.7rem;
  margin-top: 2rem;
}

.copyright a {
  color: #f4dc26;
}

.timer-display {
  position: fixed;
  top: 1rem;
  right: 1rem;
  background-color: rgba(0, 0, 0, 0.6);
  color: #fff;
  font-size: 1.25rem;
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  z-index: 10;
  font-family: "Press Start 2P", cursive;
}

.back-button {
  position: fixed;
  top: 1rem;
  left: 1rem;
  background: transparent;
  color: var(--light);
  border: 1px solid var(--light);
  padding: 0.5rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  z-index: 1000;
  font-family: var(--font);
  transition: all 0.3s ease;
}
.back-button:hover {
  background-color: var(--light);
  color: var(--dark);
}

.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.popup {
  background: #2b2b2b;
  border: 2px solid #ffffff; /* viền trắng */
  border-radius: 16px;
  padding: 2rem 1rem;
  font-family: "Press Start 2P", monospace;
  text-align: center;
  color: var(--light);
  max-width: 670px;
  width: 100%;
  box-shadow: 0 0 12px rgba(255, 255, 255, 0.6),
    /* ánh sáng trắng gần */ 0 0 24px rgba(255, 255, 255, 0.3); /* ánh sáng trắng xa hơn */
  animation: fadeIn 0.3s ease;
}

.popup-message {
  font-size: 0.9rem;
  margin-bottom: 1rem;
  line-height: 1.5;
  letter-spacing: 1px;
}

.popup-buttons button {
  font-family: "Press Start 2P", cursive;
  font-size: 0.75rem;
  margin: 0 1rem;
  padding: 0.7rem 1.3rem;
  border: 1px solid var(--light);
  background: transparent;
  color: var(--light);
  cursor: pointer;
  border-radius: 8px;
  transition: 0.3s;
}

.popup-buttons button:hover {
  background: var(--light);
  color: var(--dark);
}
</style>
