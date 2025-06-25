<template>
  <div class="tic-tac-toe-app">
    <header class="header">
      <h1 class="game-title">Tic Tac Toe</h1>
      <button
        class="start-btn"
        @click="resetGame"
        :aria-label="'Start new game / Reset board'"
        data-testid="reset-game"
      >
        New Game
      </button>
    </header>

    <main class="main-board">
      <div class="board-container">
        <div
          class="tic-tac-toe-board"
          role="grid"
          aria-label="Tic Tac Toe board"
        >
          <button
            v-for="(cell, idx) in board"
            :key="idx"
            class="cell"
            :class="{ 'cell--win': winnerLine.includes(idx), 'cell--filled': cell !== '' }"
            :aria-label="getCellAriaLabel(idx)"
            :disabled="cell !== '' || winner !== null"
            @click="makeMove(idx)"
            data-testid="cell"
          >
            <span class="cell-content">{{ cell }}</span>
          </button>
        </div>
        <div class="message" :style="{ color: messageColor }" data-testid="game-message">
          <span v-if="!!winner && winner !== 'draw'">🎉 Player <span :style="{color: winnerColor}">{{ winner }}</span> wins!</span>
          <span v-else-if="winner === 'draw'">It's a draw!</span>
          <span v-else>Player <span :style="{color: turnColor}">{{ currentPlayer }}</span>'s turn</span>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
// PUBLIC_INTERFACE
import { ref, computed } from 'vue'

/**
 * State and logic for Tic Tac Toe Game.
 * Features:
 * - Interactive 3x3 board
 * - Player turn indication (X, O)
 * - Start new game/reset
 * - Win/draw detection
 * - Responsive, modern, minimal, light-themed UI
 */

// Define constants for theme
const COLORS = {
  primary: '#1976d2',
  accent: '#ff7043',
  secondary: '#ffffff'
}

type Player = 'X' | 'O'

const emptyBoard = () => Array(9).fill('')

const board = ref<string[]>(emptyBoard())
const currentPlayer = ref<Player>('X')
const winner = ref<null | Player | 'draw'>(null)
const winnerLine = ref<number[]>([])

const turnColor = computed(() =>
  currentPlayer.value === 'X' ? COLORS.primary : COLORS.accent
)
const winnerColor = computed(() =>
  winner.value === 'X' ? COLORS.primary : COLORS.accent
)
const messageColor = computed(() =>
  winner.value === 'draw'
    ? '#888'
    : (winner.value === 'X'
      ? COLORS.primary
      : (winner.value === 'O' ? COLORS.accent : '#222'))
)

/**
 * PUBLIC_INTERFACE
 * Make move for current player, check win/draw.
 */
function makeMove(idx: number) {
  if (board.value[idx] !== '' || winner.value) return
  board.value[idx] = currentPlayer.value
  checkGameResult()
  if (!winner.value) {
    currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
  }
}

/**
 * PUBLIC_INTERFACE
 * Start a new game/reset the board.
 */
function resetGame() {
  board.value = emptyBoard()
  currentPlayer.value = 'X'
  winner.value = null
  winnerLine.value = []
}

/**
 * PUBLIC_INTERFACE
 * Check for a win or draw and update state.
 */
function checkGameResult() {
  const lines = [
    [0,1,2],[3,4,5],[6,7,8], // rows
    [0,3,6],[1,4,7],[2,5,8], // columns
    [0,4,8],[2,4,6]          // diagonals
  ]
  for (const line of lines) {
    const [a,b,c] = line
    if (
      board.value[a] &&
      board.value[a] === board.value[b] &&
      board.value[a] === board.value[c]
    ) {
      winner.value = board.value[a] as Player
      winnerLine.value = line
      return
    }
  }
  if (board.value.every(cell => cell !== '')) {
    winner.value = 'draw'
    winnerLine.value = []
  }
}

/**
 * PUBLIC_INTERFACE
 * Get ARIA label for a board cell for accessibility.
 */
function getCellAriaLabel(idx: number) {
  const v = board.value[idx]
  if (v === '') return `Empty cell ${idx + 1}. Click to make a move.`
  return `Cell ${idx + 1}, occupied by ${v}`
}
</script>

<style scoped>
:root {
  --primary: #1976d2;
  --accent: #ff7043;
  --secondary: #ffffff;
}
.tic-tac-toe-app {
  min-height: 100vh;
  background: var(--secondary);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  box-sizing: border-box;
  padding: 0 16px;
}
.header {
  width: 100%;
  max-width: 420px;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 40px auto 0;
  gap: 14px;
}
.game-title {
  font-size: 2.4rem;
  font-weight: 700;
  color: var(--primary);
  text-align: center;
  letter-spacing: 0.02em;
  margin-bottom: 0.1em;
}
.start-btn {
  padding: 0.5em 1.8em;
  border-radius: 100px;
  border: none;
  background: var(--primary);
  color: var(--secondary);
  font-size: 1.09rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}
.start-btn:hover,
.start-btn:focus-visible {
  background: var(--accent);
  outline: none;
}
.main-board {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
}
.board-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 28px;
  margin-bottom: 36px;
  width: 100%;
}
.tic-tac-toe-board {
  display: grid;
  grid-template-columns: repeat(3, 62px);
  grid-template-rows: repeat(3, 62px);
  gap: 8px;
  background: #f5f8fb;
  padding: 17px;
  border-radius: 16px;
  box-shadow: 0 4px 18px 0 rgba(66, 133, 244, 0.09), 0 1.5px 6px 0 rgba(0,0,0,0.08);
  justify-content: center;
  margin-bottom: 18px;
  width: max-content;
}
.cell {
  background: var(--secondary);
  border: 2.4px solid #e3eaf6;
  border-radius: 12px;
  font-size: 2.3rem;
  font-weight: 600;
  color: #303859;
  cursor: pointer;
  transition: border 0.2s, background 0.1s;
  outline: none;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 62px;
  width: 62px;
  position: relative;
  user-select: none;
}
.cell--win {
  border-color: var(--accent);
  background: #fff8f4;
}
.cell--filled {
  cursor: default;
  color: #757575;
}
.cell:active:not(:disabled) {
  background: #e8ecfa;
}
.cell:focus-visible {
  border-color: var(--primary);
  outline: 2px solid var(--primary);
}
.cell-content {
  pointer-events: none;
}
.message {
  min-height: 34px;
  font-size: 1.22rem;
  font-weight: 500;
  text-align: center;
  margin-top: 10px;
  letter-spacing: 0.01em;
}
@media (max-width: 600px) {
  .header {
    margin-top: 18px;
    max-width: 95vw;
  }
  .game-title {
    font-size: 1.45rem;
  }
  .board-container {
    margin-top: 12px;
    margin-bottom: 20px;
  }
  .tic-tac-toe-board {
    grid-template-columns: repeat(3, 18vw);
    grid-template-rows: repeat(3, 18vw);
    gap: 2.7vw;
    padding: 4vw;
    border-radius: 8vw;
    width: 100%;
    min-width: unset;
    box-shadow: 0 2px 9px 0 rgba(66,133,244,0.10), 0 1.2px 4px rgba(0,0,0,0.09);
  }
  .cell {
    font-size: 1.9rem;
    border-radius: 4vw;
    height: 18vw;
    width: 18vw;
  }
}
</style>
