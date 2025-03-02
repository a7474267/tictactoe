<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { WORD_LIST } from "../constants/wordList";
import Fireworks from "./Fireworks.vue";

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

// 提示訊息
const toast = ref("");
const isGameOverToast = ref(false);

const showToast = (message, isGameOver = false) => {
  toast.value = message;
  isGameOverToast.value = isGameOver;
  if (!isGameOver) {
    setTimeout(() => {
      toast.value = "";
      isGameOverToast.value = false;
    }, 2000);
  }
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
    showToast("這個單字不在題庫中！");
    return;
  }

  if (currentGuess.value === SOLUTION.value) {
    gameStatus.value = "won";
    // 延遲一下再重置遊戲，讓玩家看到勝利的效果
    setTimeout(() => {
      resetGame();
    }, 2000);
  } else if (currentAttempt.value === MAX_ATTEMPTS - 1) {
    gameStatus.value = "lost";
    showToast(`正確答案是：${SOLUTION.value}`, true);
  }

  // 只有在遊戲還在進行中時才增加嘗試次數並清空當前猜測
  if (gameStatus.value === "playing") {
    currentAttempt.value++;
    currentGuess.value = "";
    updateCurrentRow("");
  }
};

// 重置遊戲
const resetGame = () => {
  board.value = initializeBoard();
  currentAttempt.value = 0;
  currentGuess.value = "";
  gameStatus.value = "playing";
  toast.value = "";
  isGameOverToast.value = false;
  // 重新計算 SOLUTION
  SOLUTION.value = getNewWord();
};

// 更新當前行
const updateCurrentRow = (guess) => {
  const row = Array(WORD_LENGTH).fill("");
  const guessLength = Math.min(guess.length, WORD_LENGTH);
  for (let i = 0; i < guessLength; i++) {
    row[i] = guess[i];
  }
  board.value[currentAttempt.value] = row;
};

// 檢查字母狀態
const getLetterStatus = (letter, position, attemptIndex) => {
  if (attemptIndex > currentAttempt.value || !letter) return "";
  if (attemptIndex === currentAttempt.value) return "";

  const guess = board.value[attemptIndex].join("");
  const solution = SOLUTION.value;

  // 如果字母在正確位置
  if (letter === solution[position]) {
    return "correct";
  }

  // 計算答案中剩餘的字母數量
  const letterCount = {};
  for (let i = 0; i < solution.length; i++) {
    if (i !== position || solution[i] !== guess[i]) {
      letterCount[solution[i]] = (letterCount[solution[i]] || 0) + 1;
    }
  }

  // 檢查之前的位置是否已經使用了這個字母
  for (let i = 0; i < position; i++) {
    if (guess[i] === letter && guess[i] !== solution[i]) {
      letterCount[letter] = (letterCount[letter] || 0) - 1;
    }
  }

  // 如果字母在答案中且還有剩餘數量
  if (letterCount[letter] > 0) {
    return "present";
  }

  return "absent";
};

// 獲取鍵盤按鈕狀態
const getKeyStatus = (key) => {
  let finalStatus = "";

  // 檢查所有已完成的嘗試
  for (let i = 0; i < currentAttempt.value; i++) {
    const row = board.value[i];
    for (let j = 0; j < row.length; j++) {
      if (row[j] === key) {
        const status = getLetterStatus(key, j, i);
        // 優先級：correct > present > absent
        if (status === "correct") {
          return "correct";
        } else if (status === "present") {
          finalStatus = "present";
        } else if (status === "absent" && !finalStatus) {
          finalStatus = "absent";
        }
      }
    }
  }

  return finalStatus;
};

// 檢查單字是否在題庫中
const isValidWord = (word) => {
  return WORD_LIST.includes(word);
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
      </div>

      <Fireworks :is-active="gameStatus === 'won'" />

      <div v-if="toast" class="toast" :class="{ 'game-over': isGameOverToast }">
        {{ toast }}
        <button
          v-if="isGameOverToast"
          class="restart-button"
          @click="resetGame"
        >
          重新開始
        </button>
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
  pointer-events: none;
}

.toast:not(.game-over) {
  animation: toast-in-out 2s ease-in-out;
}

.toast.game-over {
  pointer-events: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 16px 32px;
}

.restart-button {
  background-color: #538d4e;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 16px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: bold;
  transition: background-color 0.2s;
}

.restart-button:hover {
  background-color: #62a65c;
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
