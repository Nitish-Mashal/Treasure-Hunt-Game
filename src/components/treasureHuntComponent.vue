<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="container text-center">
      <h2 class="text-2xl font-bold mb-10 pt-10">Seek Game</h2>

      <!-- Grid -->
      <div class="grid grid-cols-5 gap-2 justify-center mx-auto" v-if="flatGrid.length">
        <button v-for="(cell, index) in flatGrid" :key="index" @click="openCell(index)" :class="[
          'w-16 h-16 rounded shadow text-xs font-semibold flex items-center justify-center transition',
          openedCells.includes(index)
            ? cell === null
              ? 'brick-bg text-white'
              : cell === 0
                ? 'bg-yellow-500 text-black'
                : 'bg-green-600 text-white'
            : 'bg-gray-700 text-white',
          'hover:scale-105'
        ]">
          {{ openedCells.includes(index) ? (cell === null ? '' : cell === 0 ? '🎉' : cell) : '' }}
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
          "http://147.93.106.108:5100/gameplays/seek/get-game?seek=683c41fec2f91538618dcce8"
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
      } else if (cellValue === 0) {
        this.showToast("🎉 Treasure Found!");
        this.launchConfetti();
      } else {
        this.showToast("Clue unlocked");
      }
    },
    showToast(message) {
      this.toastMessage = message;
      setTimeout(() => {
        this.toastMessage = "";
      }, 2000);
    },
    launchConfetti() {
      confetti({
        particleCount: 150,
        spread: 100,
        origin: { y: 0.6 },
      });
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
</style>
