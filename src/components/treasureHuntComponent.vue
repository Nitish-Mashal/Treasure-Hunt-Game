<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100 relative">

    <!-- Game Icon (fixed top-left) -->
    <div class="absolute top-4 left-4">
      <img src="/seek.png" alt="Game Icon" class="w-12 h-12">
    </div>

    <div class="container text-center pb-2">

      <!-- Grid -->
      <div class="grid grid-cols-5 gap-2 justify-center mx-auto" v-if="flatGrid.length">
        <button v-for="(cell, index) in flatGrid" :key="index" @click="openCell(index)" :class="[
          'w-16 h-16 rounded shadow text-xl font-semibold flex items-center justify-center transition',
          openedCells.includes(index)
            ? cell === null
              ? 'brick-bg text-white'
              : cell === 0
                ? 'bg-yellow-500 text-black'
                : 'bg-green-600 text-white'
            : 'bg-gray-700 text-white',
          'hover:scale-105'
        ]">
          {{ openedCells.includes(index) ? (cell === null ? '' : cell === 0 ? '💎' : cell) : '' }}
        </button>
      </div>
    </div>

    <!-- Toast Message -->
    <div v-if="toastMessage"
      class="fixed bottom-6 left-1/2 transform -translate-x-1/2 bg-black bg-opacity-80 text-white text-sm px-4 py-2 rounded shadow-lg z-50 select-none">
      {{ toastMessage }}
    </div>
  </div>
</template>

<script>
import axios from "axios";
import confetti from "canvas-confetti";

export default {
  data() {
    return {
      openedCells: [],
      toastMessage: "",
      rows: 0,
      columns: 0,
      grid: [],
    };
  },
  computed: {
    flatGrid() {
      return this.grid.flat();
    },
  },
  methods: {
    async fetchGameData() {
      try {
        const response = await axios.get(
          "https://aqada.online/gameplays/seek/get-game?seek=683c41fec2f91538618dcce8"
        );
        const data = response.data;

        this.grid = data.clues;
        this.rows = data.grid_rows;
        this.columns = data.grid_cols;
      } catch (error) {
        console.error("Failed to load game data:", error);
        this.showToast("Failed to load game data");
      }
    },
    openCell(index) {
      if (this.openedCells.includes(index)) return;

      this.openedCells.push(index);
      const cellValue = this.flatGrid[index];

      if (cellValue === null) {
        this.showToast("No clue here");
      }
      else if (cellValue === 0) {
        this.showToast("💎 Treasure Found!");
        this.launchConfetti();
      }
      else {
        this.showToast(`${cellValue} cells away`);
      }
    },
    showToast(message) {
      this.toastMessage = message;
      setTimeout(() => {
        this.toastMessage = "";
      }, 2000);
    },
    launchConfetti() {
      const duration = 4000; // 4 seconds
      const end = Date.now() + duration;

      const interval = setInterval(() => {
        confetti({
          particleCount: 80,
          spread: 100,
          origin: { y: 0.6 }
        });

        if (Date.now() > end) {
          clearInterval(interval);
        }
      }, 300);
    },
  },
  mounted() {
    this.fetchGameData();
  },
};
</script>

<style scoped>
.brick-bg {
  background-color: #b0a99f;
  background-image:
    linear-gradient(90deg, #a89c92 5%, transparent 5%),
    linear-gradient(#a89c92 5%, transparent 5%);
  background-size: 20px 20px;
  background-position: 0 0;
}

@keyframes pop {
  0% {
    transform: scale(0.8);
  }

  100% {
    transform: scale(1.1);
  }
}
</style>
