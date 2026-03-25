<template>
  <div class="h-screen w-screen overflow-hidden bg-gray-100">
    <div class="flex items-center justify-center h-full relative">

      <!-- Game Icon -->
      <div class="absolute top-4 left-4 z-10">
        <img src="/seek.png" class="w-12 h-12" />
      </div>

      <div class="container text-center pb-2">

        <!-- GRID -->
        <div class="grid grid-cols-5 gap-2 justify-center mx-auto" v-if="flatGrid.length">
          <button v-for="(cell, index) in flatGrid" :key="index" @click="openCell(index)" :class="[
            'w-16 h-16 rounded shadow text-xl font-semibold flex items-center justify-center transition',
            openedCells.includes(index)
              ? cell === null
                ? 'brick-bg text-white'
                : Number(cell) === 0
                  ? 'bg-yellow-500 text-black'
                  : 'bg-green-600 text-white'
              : 'bg-gray-700 text-white'
          ]">
            {{
              openedCells.includes(index)
                ? cell === null
                  ? ''
                  : Number(cell) === 0
                    ? '💎'
                    : cell
                : ''
            }}
          </button>
        </div>

        <div class="mt-4 text-lg font-semibold">
          Tries: {{ tries }}
        </div>

      </div>

      <!-- Toast -->
      <div v-if="toastMessage" class="fixed bottom-6 left-1/2 -translate-x-1/2 bg-black text-white px-4 py-2 rounded">
        {{ toastMessage }}
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue"
import axios from "axios"
import confetti from "canvas-confetti"

/* STATE */
const openedCells = ref([])
const grid = ref([])
const tries = ref(0)
const toastMessage = ref("")

const startTime = ref(null)
const elapsedSeconds = ref(0)
let timerInterval = null

const gameCompleted = ref(false)

const userId = ref(null)
const seekId = ref(null)
const gameId = ref(null)

/* GRID */
const flatGrid = computed(() => grid.value.flat())

/* ✅ EXTRACT PARAMS (FIXED) */
const extractUrlParams = () => {
  const params = new URLSearchParams(window.location.search)

  const seek = params.get("seek")
  const user = params.get("user")
  const game_id = params.get("game_id")

  console.log("✅ URL Params:", { seek, user, game_id })

  return { seek, user, game_id }
}

/* TOAST */
function showToast(msg) {
  toastMessage.value = msg
  setTimeout(() => (toastMessage.value = ""), 2000)
}

/* CONFETTI */
function launchConfetti() {
  const end = Date.now() + 3000
  const interval = setInterval(() => {
    confetti({ particleCount: 70, spread: 90, origin: { y: 0.6 } })
    if (Date.now() > end) clearInterval(interval)
  }, 300)
}

/* TIMER (RESUME LOGIC) */
function startTimer(id) {
  const key = `seek_timer_${id}`

  const storedStart = localStorage.getItem(key)

  if (storedStart) {
    startTime.value = parseInt(storedStart)
  } else {
    startTime.value = Date.now()
    localStorage.setItem(key, startTime.value)
  }

  timerInterval = setInterval(() => {
    elapsedSeconds.value = Math.floor(
      (Date.now() - startTime.value) / 1000
    )
  }, 1000)
}

/* FETCH GAME */
async function fetchGameData(id) {
  try {
    const res = await axios.get(
      "https://aqada.online/gameplays/seek/get-game",
      { params: { seek: id } }
    )

    grid.value = res.data.clues || []
    console.log("✅ Grid Loaded:", grid.value)

  } catch (e) {
    console.error("❌ Fetch Error:", e)
    showToast("Game load failed")
  }
}

/* OPEN CELL */
async function openCell(index) {

  // 🚫 Stop after completion
  if (gameCompleted.value) return

  if (openedCells.value.includes(index)) return

  tries.value++
  openedCells.value.push(index)

  const cellValue = flatGrid.value[index]

  console.log("Clicked:", cellValue, typeof cellValue)

  if (cellValue === null) {
    showToast("No clue here")
    return
  }

  /* ✅ TREASURE FOUND FIXED CONDITION */
  if (Number(cellValue) === 0) {

    console.log("🎯 Treasure Found Triggered")

    gameCompleted.value = true

    showToast("💎 Treasure Found!")
    launchConfetti()

    clearInterval(timerInterval)

    if (!userId.value) {
      console.error("❌ User missing in URL")
      return
    }

    try {
      console.log("🚀 Calling game-completed API...")

      const formData = new URLSearchParams()

      formData.append("game_id", gameId.value || seekId.value)
      formData.append("user", userId.value)

      const payload = {
        no_of_cells: openedCells.value.length,
        seconds: elapsedSeconds.value
      }

      formData.append("params", JSON.stringify(payload))

      console.log("📤 Payload:", payload)

      const response = await axios.post(
        "https://aqada.online/games/game-completed",
        formData,
        {
          headers: {
            "Content-Type": "application/x-www-form-urlencoded"
          }
        }
      )

      console.log("✅ API Success:", response.data)

      localStorage.removeItem(`seek_timer_${seekId.value}`)

    } catch (err) {
      console.error("❌ API Error:", err)
    }

  } else {
    showToast(`${cellValue} cells away`)
  }
}

/* INIT */
onMounted(() => {
  const params = extractUrlParams()

  seekId.value = params.seek
  userId.value = params.user
  gameId.value = params.game_id || params.seek

  if (!seekId.value) {
    console.error("❌ Seek ID missing")
    return
  }

  fetchGameData(seekId.value)
  startTimer(seekId.value)
})
</script>