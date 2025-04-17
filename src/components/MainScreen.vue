<template>
  <div class="screen">
    <h1>Mew Mew Memories</h1>
    <p>Select mode to start game</p>
    <div class="modes">
      <button @click="handleClick(16)">
        <span>4x4</span>
        <span>Easy</span>
      </button>
      <button @click="handleClick(36)">
        <span>6x6</span>
        <span>Normal</span>
      </button>
      <button @click="handleClick(64)">
        <span>8x8</span>
        <span>Hard</span>
      </button>
    </div>

    <audio ref="bgm" src="/sounds/bgMusic.mp3" autoplay loop></audio>
    <!-- Slider điều chỉnh âm lượng -->
    <div class="volume-control">
      <label for="volume">Music Volume</label>
      <input
        id="volume"
        type="range"
        min="0"
        max="1"
        step="0.01"
        v-model.number="bgVolume"
        @input="updateVolume"
      />
    </div>
  </div>
</template>

<script>
export default {
  emits: ["onStart"],
  data() {
    return {
      bgVolume: 1, // volume mặc định = 100%
    };
  },
  mounted() {
    const bgm = this.$refs.bgm;
    if (bgm) {
      // Reset về đầu mỗi lần vào màn hình chính
      bgm.currentTime = 0;

      // Chờ người dùng click để phát
      const enableAudio = () => {
        bgm.volume = this.bgVolume;
        bgm.play().catch(() => {});
        document.removeEventListener("click", enableAudio);
      };

      document.addEventListener("click", enableAudio);
    }
  },
  beforeUnmount() {
    const bgm = this.$refs.bgm;
    if (bgm) {
      bgm.pause(); // Dừng nhạc
      // Không cần reset currentTime
    }
  },
  methods: {
    updateVolume() {
      const bgm = this.$refs.bgm;
      if (bgm) {
        bgm.volume = this.bgVolume;
      }
    },

    handleClick(totalOfBlocks) {
      const audio = new Audio(`/sounds/click.mp3`);
      audio.volume = 0.5;
      audio.play();
      // Khi âm thanh click kết thúc → vào màn chơi
      audio.addEventListener("ended", () => {
        this.onStart(totalOfBlocks);
      });
      setTimeout(() => {
        this.onStart(totalOfBlocks); // Gọi lại onStart gốc
      }, 700); // delay nhẹ để không cắt âm click
    },
    onStart(totalOfBlocks) {
      this.$emit("onStart", { totalOfBlocks });
    },
  },
};
</script>

<style lang="css" scoped>
.screen {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  background-color: var(--dark);
  color: var(--light);
}

.screen h1 {
  font-size: 4.5rem;
  text-transform: uppercase;
}

.screen p {
  font-size: 2rem;
}

.modes {
  display: flex;
  margin-top: 2rem;
}

.modes button {
  font: var(--font);
  width: 150px;
  height: 150px;
  background: rgba(255, 255, 255, 0.05); /* nền nhẹ nhẹ */
  border: 2px solid var(--light);
  color: var(--light);
  display: flex;
  flex-direction: column;
  border-radius: 1rem;
  margin: 0 1rem;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(4px); /* hiệu ứng kính mờ */
  position: relative;
  overflow: hidden;
}

.modes button::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.2) 10%,
    transparent 60%
  );
  transform: rotate(25deg);
  opacity: 0;
  transition: opacity 0.4s;
  pointer-events: none;
}

.modes button:hover::before {
  opacity: 1;
}

.modes button:hover {
  transform: scale(1.05);
  background-color: var(--light);
  color: var(--dark);
  box-shadow: 0 0 15px var(--light), 0 0 30px var(--light);
}

.modes button span:first-child {
  font-size: 2rem;
}

.modes button span:last-child {
  display: block;
  font-size: 1.25rem;
  margin-top: 0.5rem;
}

.volume-control {
  margin-top: 2rem;
  color: var(--light);
  text-align: center;
  font-size: 1rem;
}

.volume-control input[type="range"] {
  width: 200px;
  margin-left: 1rem;
}
</style>
