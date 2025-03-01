<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";

// 單字庫
const WORD_LIST = [
  "APPLE",
  "BEACH",
  "CLOUD",
  "DREAM",
  "EARTH",
  "FLAME",
  "GRAPE",
  "HEART",
  "IMAGE",
  "JUICE",
  "KNIFE",
  "LIGHT",
  "MUSIC",
  "NIGHT",
  "OCEAN",
  "PEACE",
  "QUEEN",
  "RIVER",
  "SMILE",
  "TIGER",
  "UNITY",
  "VOICE",
  "WATER",
  "YOUTH",
  "ZEBRA",
];

// 根據日期獲取題目
const getDailyWord = () => {
  const today = new Date();
  const dateString = `${today.getFullYear()}-${
    today.getMonth() + 1
  }-${today.getDate()}`;
  let total = 0;
  for (let i = 0; i < dateString.length; i++) {
    total += dateString.charCodeAt(i);
  }
  return WORD_LIST[total % WORD_LIST.length];
};

// 遊戲配置
const WORD_LENGTH = 5;
const MAX_ATTEMPTS = 6;
const SOLUTION = computed(() => getDailyWord());

// 初始化二維數組
const initializeBoard = () => {
  return Array(MAX_ATTEMPTS)
    .fill()
    .map(() => Array(WORD_LENGTH).fill(""));
};

// 遊戲狀態
const board = ref(initializeBoard());
const currentAttempt = ref(0);
const currentGuess = ref("");
const gameStatus = ref("playing"); // playing, won, lost

// 上次遊戲日期
const lastPlayedDate = ref(localStorage.getItem("lastPlayedDate") || "");
const todayDate = new Date().toDateString();

// 檢查是否需要重置遊戲
const checkAndResetGame = () => {
  if (lastPlayedDate.value !== todayDate) {
    resetGame();
    localStorage.setItem("lastPlayedDate", todayDate);
    lastPlayedDate.value = todayDate;
  }
};

// 煙火特效配置
const fireworks = ref([]);
const maxFireworks = 30;

// 創建煙火
const createFirework = () => {
  if (fireworks.value.length >= maxFireworks) return;

  const firework = {
    id: Date.now() + Math.random(),
    x: Math.random() * 100,
    y: Math.random() * 40 + 30,
    color: `hsl(${Math.random() * 360}, 80%, 60%)`,
    tx: (Math.random() - 0.5) * 200,
    ty: (Math.random() - 0.5) * 100 - 50,
    rotate: Math.random() * 360,
    distance: 40 + Math.random() * 40,
    scale: 0.5 + Math.random() * 1,
  };

  fireworks.value.push(firework);
  setTimeout(() => {
    fireworks.value = fireworks.value.filter((f) => f.id !== firework.id);
  }, 1500);
};

// 啟動煙火
const startFireworks = () => {
  if (gameStatus.value !== "won") return;

  const interval = setInterval(() => {
    if (gameStatus.value !== "won") {
      clearInterval(interval);
      fireworks.value = [];
      return;
    }
    for (let i = 0; i < 3; i++) {
      setTimeout(() => createFirework(), i * 50);
    }
  }, 200);
};

// 更新當前行
const updateCurrentRow = (guess) => {
  const row = [...Array(WORD_LENGTH).fill("")];
  for (let i = 0; i < guess.length; i++) {
    row[i] = guess[i];
  }
  board.value[currentAttempt.value] = row;
};

// 檢查字母狀態
const getLetterStatus = (letter, position, attemptIndex) => {
  if (attemptIndex > currentAttempt.value || !letter) return "";
  if (attemptIndex === currentAttempt.value) return "";
  if (letter === SOLUTION.value[position]) return "correct";
  if (SOLUTION.value.includes(letter)) return "present";
  return "absent";
};

// 獲取鍵盤按鈕狀態
const getKeyStatus = (key) => {
  let status = "";
  for (let i = 0; i < currentAttempt.value; i++) {
    const row = board.value[i];
    for (let j = 0; j < row.length; j++) {
      if (row[j] === key) {
        const newStatus = getLetterStatus(key, j, i);
        if (newStatus === "correct") return "correct";
        if (newStatus === "present" && status !== "correct") status = "present";
        if (newStatus === "absent" && !status) status = "absent";
      }
    }
  }
  return status;
};

// 處理鍵盤輸入
const handleKeyInput = (key) => {
  if (gameStatus.value !== "playing") return;
  if (currentAttempt.value >= MAX_ATTEMPTS) return;

  if (key === "Backspace" || key === "Delete") {
    currentGuess.value = currentGuess.value.slice(0, -1);
    updateCurrentRow(currentGuess.value);
  } else if (key === "Enter" && currentGuess.value.length === WORD_LENGTH) {
    submitGuess();
  } else if (
    key.length === 1 &&
    key.match(/[a-z]/i) &&
    currentGuess.value.length < WORD_LENGTH
  ) {
    currentGuess.value += key.toUpperCase();
    updateCurrentRow(currentGuess.value);
  }
};

// 處理實體鍵盤事件
const handleKeyDown = (event) => {
  handleKeyInput(event.key);
};

// 監聽鍵盤事件
onMounted(() => {
  window.addEventListener("keydown", handleKeyDown);
  checkAndResetGame();
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyDown);
});

// 提交猜測
const submitGuess = () => {
  if (currentGuess.value === SOLUTION.value) {
    gameStatus.value = "won";
    startFireworks();
  } else if (currentAttempt.value === MAX_ATTEMPTS - 1) {
    gameStatus.value = "lost";
  }

  currentAttempt.value++;
  currentGuess.value = "";
  updateCurrentRow("");
};

// 重置遊戲
const resetGame = () => {
  board.value = initializeBoard();
  currentAttempt.value = 0;
  currentGuess.value = "";
  gameStatus.value = "playing";
  fireworks.value = [];
};

// 鍵盤配置
const keyboard = [
  ["Q", "W", "E", "R", "T", "Y", "U", "I", "O", "P"],
  ["A", "S", "D", "F", "G", "H", "J", "K", "L"],
  ["Enter", "Z", "X", "C", "V", "B", "N", "M", "Backspace"],
];
</script>

<template>
  <div class="page-container">
    <div class="game-container">
      <div class="game-content">
        <div class="board">
          <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
            <div
              v-for="(letter, colIndex) in row"
              :key="colIndex"
              class="cell"
              :class="[
                getLetterStatus(letter, colIndex, rowIndex),
                {
                  active: rowIndex === currentAttempt.value,
                  filled: letter !== '',
                },
              ]"
            >
              {{ letter }}
            </div>
          </div>
        </div>

        <div class="keyboard">
          <div
            v-for="(row, rowIndex) in keyboard"
            :key="rowIndex"
            class="keyboard-row"
          >
            <button
              v-for="key in row"
              :key="key"
              class="key"
              :class="[
                key === 'Enter' || key === 'Backspace' ? 'wide-key' : '',
                key.length === 1 ? getKeyStatus(key) : '',
              ]"
              @click="handleKeyInput(key)"
            >
              {{ key === "Backspace" ? "⌫" : key }}
            </button>
          </div>
        </div>

        <button
          v-if="gameStatus !== 'playing'"
          class="reset-button"
          @click="resetGame"
        >
          重新開始
        </button>
      </div>

      <div v-if="gameStatus === 'won'" class="fireworks-container">
        <div
          v-for="firework in fireworks"
          :key="firework.id"
          class="firework"
          :style="{
            left: firework.x + '%',
            top: firework.y + '%',
            backgroundColor: firework.color,
            color: firework.color,
            '--tx': firework.tx + 'px',
            '--ty': firework.ty + 'px',
            '--rotate': firework.rotate + 'deg',
            '--distance': firework.distance + 'px',
            '--scale': firework.scale,
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.page-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  width: 100%;
  margin: 0;
  padding: 0;
  background-color: #121213;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 500px;
  box-sizing: border-box;
  padding: 1rem;
  justify-content: center;
  gap: 1rem;
}

.game-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  width: 100%;
}

.daily-info {
  font-size: 1rem;
  color: #818384;
  text-align: center;
  margin-bottom: 0.5rem;
}

.status {
  font-size: 1.5rem;
  font-weight: bold;
  color: #ffffff;
  margin-bottom: 1rem;
}

.board {
  display: flex;
  flex-direction: column;
  gap: min(5px, 1vw);
  margin: 0 auto;
  width: fit-content;
}

.row {
  display: flex;
  gap: min(5px, 1vw);
}

.cell {
  width: min(52px, calc(15vw - 10px));
  height: min(52px, calc(15vw - 10px));
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: min(1.8rem, 5vw);
  font-weight: bold;
  background-color: transparent;
  border: 2px solid #3a3a3c;
  color: #ffffff;
  border-radius: 4px;
  transition: all 0.2s;
}

.cell.filled {
  border-color: #565758;
}

.cell.pop-in {
  animation: pop-in 0.1s ease-in-out;
}

@keyframes pop-in {
  0% {
    transform: scale(0.8);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.cell.correct {
  background-color: #538d4e;
  border-color: #538d4e;
}

.cell.present {
  background-color: #b59f3b;
  border-color: #b59f3b;
}

.cell.absent {
  background-color: #3a3a3c;
  border-color: #3a3a3c;
}

.keyboard {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  max-width: 500px;
  padding: 0;
  margin-bottom: 2rem;
}

.keyboard-row {
  display: flex;
  justify-content: center;
  gap: 6px;
  width: 100%;
}

.key {
  width: 43px;
  height: 58px;
  padding: 0;
  font-size: 1.2rem;
  font-weight: bold;
  background-color: #818384;
  color: #ffffff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  text-transform: uppercase;
  display: flex;
  align-items: center;
  justify-content: center;
  white-space: nowrap;
  overflow: hidden;
}

.key:hover {
  filter: brightness(1.1);
}

.key.correct {
  background-color: #538d4e;
}

.key.present {
  background-color: #b59f3b;
}

.key.absent {
  background-color: #3a3a3c;
}

.wide-key {
  width: 65px;
  font-size: 0.9rem;
  letter-spacing: -0.5px;
}

.reset-button {
  padding: 12px 24px;
  font-size: 1.2rem;
  background-color: #538d4e;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  margin-top: 1rem;
}

.reset-button:hover {
  filter: brightness(1.1);
}

.fireworks-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1000;
  overflow: hidden;
}

.firework {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  animation: explode 1.5s ease-out forwards;
  box-shadow: 0 0 10px 2px currentColor;
  transform: scale(var(--scale));
}

@keyframes explode {
  0% {
    transform: scale(0) translate(0, 100vh);
    opacity: 0;
  }
  5% {
    opacity: 1;
  }
  30% {
    transform: scale(var(--scale)) translate(var(--tx), var(--ty));
    opacity: 1;
  }
  100% {
    transform: scale(0) translate(var(--tx), var(--ty));
    opacity: 0;
  }
}

.firework::before,
.firework::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  transform-origin: -50% -50%;
  animation: sparkle 1.5s ease-out forwards;
  box-shadow: 0 0 6px 1px currentColor;
}

.firework::before {
  transform: rotate(45deg) translate(0, 0);
  background: inherit;
}

.firework::after {
  transform: rotate(-45deg) translate(0, 0);
  background: inherit;
}

@keyframes sparkle {
  0% {
    transform: scale(0) rotate(0) translate(0, 0);
    opacity: 0;
  }
  5% {
    opacity: 1;
  }
  30% {
    transform: scale(1) rotate(var(--rotate)) translate(var(--distance), 0);
    opacity: 1;
  }
  100% {
    transform: scale(0) rotate(var(--rotate)) translate(var(--distance), 0);
    opacity: 0;
  }
}

@media (max-width: 400px) {
  .cell {
    width: calc(18vw - 10px);
    height: calc(18vw - 10px);
    font-size: 1.5rem;
  }
}

@media (min-width: 768px) {
  .cell {
    width: 62px;
    height: 62px;
    font-size: 2rem;
  }
  .game-content {
    padding-bottom: 5rem;
  }

  .reset-button {
    margin-top: 3rem;
  }

  .keyboard {
    margin-bottom: 3rem;
  }
}
</style>
