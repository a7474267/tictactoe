<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { WORD_LIST } from "../constants/wordList";

// 上次遊戲日期
const lastPlayedDate = ref(localStorage.getItem("lastPlayedDate") || "");
// 已使用的單字
const usedWords = ref(JSON.parse(localStorage.getItem("usedWords") || "[]"));

// 獲取新的未使用單字
const getNewWord = () => {
  const availableWords = WORD_LIST.filter(
    (word) => !usedWords.value.includes(word)
  );
  if (availableWords.length === 0) {
    // 如果所有單字都用完了，重置已使用單字列表
    usedWords.value = [];
    localStorage.setItem("usedWords", "[]");
    return WORD_LIST[Math.floor(Math.random() * WORD_LIST.length)];
  }
  const newWord =
    availableWords[Math.floor(Math.random() * availableWords.length)];
  usedWords.value.push(newWord);
  localStorage.setItem("usedWords", JSON.stringify(usedWords.value));
  return newWord;
};

// 遊戲配置
const WORD_LENGTH = 5;
const MAX_ATTEMPTS = 6;
const todayDate = ref(new Date().toDateString());
const SOLUTION = computed(() => {
  // 確保每次日期改變時都重新計算
  return getNewWord();
});

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

// 檢查是否需要重置遊戲
const checkAndResetGame = () => {
  const currentDate = new Date().toDateString();
  if (lastPlayedDate.value !== currentDate) {
    resetGame();
    localStorage.setItem("lastPlayedDate", currentDate);
    lastPlayedDate.value = currentDate;
    todayDate.value = currentDate; // 更新當前日期，觸發 SOLUTION 重新計算
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

// 檢查單字是否在題庫中
const isValidWord = (word) => {
  return WORD_LIST.includes(word);
};

// 提示訊息
const toast = ref("");
const showToast = (message) => {
  toast.value = message;
  setTimeout(() => {
    toast.value = "";
  }, 2000);
};

// 處理鍵盤輸入
const handleKeyInput = (key) => {
  if (gameStatus.value !== "playing") return;
  if (currentAttempt.value >= MAX_ATTEMPTS) return;

  if (key === "Backspace" || key === "Delete") {
    currentGuess.value = currentGuess.value.slice(0, -1);
    updateCurrentRow(currentGuess.value);
  } else if (key === "Enter" && currentGuess.value.length === WORD_LENGTH) {
    if (!isValidWord(currentGuess.value)) {
      showToast("這個單字不在題庫中！");
      return;
    }
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

  // 每分鐘檢查一次日期是否改變
  const dateCheckInterval = setInterval(() => {
    checkAndResetGame();
  }, 60000);

  onUnmounted(() => {
    window.removeEventListener("keydown", handleKeyDown);
    clearInterval(dateCheckInterval);
  });
});

// 提交猜測
const submitGuess = () => {
  // 檢查單字是否有效
  if (!isValidWord(currentGuess.value)) {
    return;
  }

  if (currentGuess.value === SOLUTION.value) {
    gameStatus.value = "won";
    startFireworks();
    // 延遲一下再重置遊戲，讓玩家看到勝利的效果
    setTimeout(() => {
      resetGame();
    }, 2000);
  } else if (currentAttempt.value === MAX_ATTEMPTS - 1) {
    gameStatus.value = "lost";
    showToast(`正確答案是：${SOLUTION.value}`);
    // 延遲一下再重置遊戲
    setTimeout(() => {
      resetGame();
    }, 2000);
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
  // 重新計算 SOLUTION
  SOLUTION.value = getNewWord();
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

      <div v-if="toast" class="toast">
        {{ toast }}
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
  overflow-y: auto;
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
  margin: 0 auto;
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
}

.keyboard-row {
  display: flex;
  width: 100%;
  gap: 6px;
  justify-content: center;
}

.key {
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1;
  height: 58px;
  padding: 0;
  border-radius: 4px;
  background: #818384;
  color: #ffffff;
  font-weight: bold;
  cursor: pointer;
  user-select: none;
  text-transform: uppercase;
  border: none;
}

.key.wide-key {
  flex: 1.5;
}

.key.correct {
  background: #538d4e;
}

.key.present {
  background: #b59f3b;
}

.key.absent {
  background: #3a3a3c;
}

.key:hover {
  filter: brightness(1.1);
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

@media (min-width: 768px) {
  .cell {
    width: 62px;
    height: 62px;
    font-size: 2rem;
  }
}

@media (max-width: 400px) {
  .cell {
    width: calc(18vw - 10px);
    height: calc(18vw - 10px);
    font-size: 1.5rem;
  }
}

.toast {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  background-color: rgba(51, 51, 51, 0.95);
  color: #fff;
  border-radius: 4px;
  z-index: 1001;
  font-size: 1rem;
  font-weight: bold;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  animation: toast-in-out 2s ease-in-out;
  pointer-events: none;
}

@keyframes toast-in-out {
  0% {
    opacity: 0;
    transform: translate(-50%, 20px);
  }
  10% {
    opacity: 1;
    transform: translate(-50%, 0);
  }
  90% {
    opacity: 1;
    transform: translate(-50%, 0);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -20px);
  }
}
</style>
