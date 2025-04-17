<template>
  <div class="screen">
    <div
      class="screen__inner"
      :style="{
        gridTemplateColumns: `repeat(${gridSize}, 150px)`,
      }"
    >
      <card-flip
        v-for="(card, index) in cardsContext"
        :key="index"
        :ref="`card-${index}`"
        :imgBackFaceUrl="`images/${card}.jpg`"
        :card="{ index, value: card }"
        :rules="rules"
        @onFlip="checkRule($event)"
      />
    </div>
  </div>
</template>

<script>
import CardFlip from "./CardFlip.vue";
export default {
  props: {
    cardsContext: {
      type: Array,
      default: function () {
        return [];
      },
      required: true,
    },
  },
  components: {
    CardFlip: CardFlip,
  },
  data() {
    return {
      rules: [],
      isChecking: false,
      totalCorrectPairs: 0,
    };
  },

  computed: {
    gridSize() {
      const length = this.cardsContext.length;
      if (length === 16) return 4;
      if (length === 64) return 8;
      if (length === 100) return 10;
      return Math.ceil(Math.sqrt(length));
    },
  },

  methods: {
    checkRule(card) {
      if (this.isChecking || this.rules.length === 2) return;
      this.rules.push(card);

      if (this.rules.length === 2) {
        this.isChecking = true;

        const [first, second] = this.rules;
        if (first.value === second.value) {
          console.log("Right...");
          this.$refs[`card-${first.index}`][0].onEnabledDisabledMode();
          this.$refs[`card-${second.index}`][0].onEnabledDisabledMode();
          this.rules = [];
          this.totalCorrectPairs++;

          if (this.totalCorrectPairs === this.cardsContext.length / 2) {
            setTimeout(() => {
              this.$emit("onFinish");
            }, 1000);
          }

          this.isChecking = false;
        } else {
          console.log("Wrong...");
          setTimeout(() => {
            this.$refs[`card-${first.index}`][0].onFlipBackCard();
            this.$refs[`card-${second.index}`][0].onFlipBackCard();
            this.rules = [];
            this.isChecking = false;
          }, 600);
        }
      }
    },
  },
};
</script>

<style scoped>
.screen {
  width: 100%;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
  background-color: var(--dark);
  color: var(--light);
}

.screen__inner {
  display: grid;
  gap: 16px;
  justify-content: center;
  flex-wrap: wrap;
  margin: 2rem auto;
  background-color: var(--dark);
}
</style>
