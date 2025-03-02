<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { WORD_LIST } from "../constants/wordList";
import Fireworks from "./Fireworks.vue";
import Toast from "./Toast.vue";
import GameBoard from "./GameBoard.vue";
import GameKeyboard from "./GameKeyboard.vue";

const lastPlayedDate = ref(localStorage.getItem("lastPlayedDate") || "");
const usedWords = ref(JSON.parse(localStorage.getItem("usedWords") || "[]"));

const getNewWord = () => {
  const availableWords = WORD_LIST.filter(
    (word) => !usedWords.value.includes(word)
  );
  if (availableWords.length === 0) {
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

const WORD_LENGTH = 5;
const MAX_ATTEMPTS = 6;
const todayDate = ref(new Date().toDateString());
const SOLUTION = ref(getNewWord());
const dateCheckInterval = ref(null);

const initializeBoard = () => {
  return Array(MAX_ATTEMPTS)
    .fill()
    .map(() => Array(WORD_LENGTH).fill(""));
};

const board = ref(initializeBoard());
const currentAttempt = ref(0);
const currentGuess = ref("");
const gameStatus = ref("playing");

const letterStatuses = ref(
  Array(MAX_ATTEMPTS)
    .fill()
    .map(() => Array(WORD_LENGTH).fill(""))
);
const keyStatuses = ref({});

const checkAndResetGame = () => {
  const currentDate = new Date().toDateString();
  if (lastPlayedDate.value !== currentDate) {
    resetGame();
    localStorage.setItem("lastPlayedDate", currentDate);
    lastPlayedDate.value = currentDate;
    todayDate.value = currentDate;
    SOLUTION.value = getNewWord();
  }
};

const toastMessage = ref("");
const isGameOverToast = ref(false);
const showToastFlag = ref(false);

const showToast = (message, isGameOver = false) => {
  toastMessage.value = message;
  isGameOverToast.value = isGameOver;
  showToastFlag.value = true;
  if (!isGameOver) {
    setTimeout(() => {
      showToastFlag.value = false;
      toastMessage.value = "";
      isGameOverToast.value = false;
    }, 2000);
  }
};

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

const handleKeyDown = (event) => {
  handleKeyInput(event.key);
};

const forceResetGame = () => {
  SOLUTION.value = getNewWord();
  board.value = initializeBoard();
  currentAttempt.value = 0;
  currentGuess.value = "";
  gameStatus.value = "playing";
  toastMessage.value = "";
  isGameOverToast.value = false;
  showToastFlag.value = false;
  letterStatuses.value = Array(MAX_ATTEMPTS)
    .fill()
    .map(() => Array(WORD_LENGTH).fill(""));
  keyStatuses.value = {};
};

onMounted(() => {
  window.addEventListener("keydown", handleKeyDown);
  forceResetGame();

  const now = new Date();
  const nextHour = new Date(now);
  nextHour.setHours(nextHour.getHours() + 1, 0, 0, 0);
  const timeUntilNextHour = nextHour - now;

  const initialTimer = setTimeout(() => {
    checkAndResetGame();
    dateCheckInterval.value = setInterval(checkAndResetGame, 3600000);
  }, timeUntilNextHour);

  onUnmounted(() => {
    window.removeEventListener("keydown", handleKeyDown);
    clearTimeout(initialTimer);
    if (dateCheckInterval.value) {
      clearInterval(dateCheckInterval.value);
    }
  });
});

const submitGuess = () => {
  if (!isValidWord(currentGuess.value)) {
    showToast("這個單字不在題庫中！");
    return;
  }

  const currentRow = currentAttempt.value;
  const guess = currentGuess.value;
  const solution = SOLUTION.value;

  const statuses = Array(WORD_LENGTH).fill("absent");
  const solutionLetters = {};

  [...solution].forEach((letter) => {
    solutionLetters[letter] = (solutionLetters[letter] || 0) + 1;
  });

  for (let i = 0; i < WORD_LENGTH; i++) {
    const letter = guess[i];
    if (letter === solution[i]) {
      statuses[i] = "correct";
      solutionLetters[letter]--;
      keyStatuses.value[letter] = "correct";
    }
  }

  for (let i = 0; i < WORD_LENGTH; i++) {
    const letter = guess[i];
    if (statuses[i] === "correct") continue;

    if (solutionLetters[letter] && solutionLetters[letter] > 0) {
      statuses[i] = "present";
      solutionLetters[letter]--;

      if (keyStatuses.value[letter] !== "correct") {
        keyStatuses.value[letter] = "present";
      }
    } else {
      if (
        keyStatuses.value[letter] !== "correct" &&
        keyStatuses.value[letter] !== "present"
      ) {
        keyStatuses.value[letter] = "absent";
      }
    }
  }

  letterStatuses.value[currentRow] = statuses;

  if (currentGuess.value === SOLUTION.value) {
    gameStatus.value = "won";
    showToast("恭喜您猜對了！", true);
  } else if (currentAttempt.value === MAX_ATTEMPTS - 1) {
    gameStatus.value = "lost";
    showToast(`正確答案是：${SOLUTION.value}`, true);
  }

  if (gameStatus.value === "playing") {
    currentAttempt.value++;
    currentGuess.value = "";
    updateCurrentRow("");
  }
};

const resetGame = () => {
  SOLUTION.value = getNewWord();
  board.value = initializeBoard();
  currentAttempt.value = 0;
  currentGuess.value = "";
  gameStatus.value = "playing";
  toastMessage.value = "";
  isGameOverToast.value = false;
  showToastFlag.value = false;
  letterStatuses.value = Array(MAX_ATTEMPTS)
    .fill()
    .map(() => Array(WORD_LENGTH).fill(""));
  keyStatuses.value = {};
};

const updateCurrentRow = (guess) => {
  const row = Array(WORD_LENGTH).fill("");
  const guessLength = Math.min(guess.length, WORD_LENGTH);
  for (let i = 0; i < guessLength; i++) {
    row[i] = guess[i];
  }
  board.value[currentAttempt.value] = row;
};

const getLetterStatus = (letter, position, attemptIndex) => {
  if (attemptIndex > currentAttempt.value || !letter) return "";
  if (attemptIndex === currentAttempt.value) return "";
  return letterStatuses.value[attemptIndex][position];
};

const getKeyStatus = (key) => {
  return keyStatuses.value[key] || "";
};

const isValidWord = (word) => {
  return WORD_LIST.includes(word);
};

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
        <GameBoard
          :board="board"
          :current-attempt="currentAttempt"
          :get-letter-status="getLetterStatus"
        />

        <GameKeyboard
          :keyboard="keyboard"
          :get-key-status="getKeyStatus"
          @key-input="handleKeyInput"
        />
      </div>

      <Fireworks :is-active="gameStatus === 'won'" />

      <Toast
        :show="showToastFlag"
        :message="toastMessage"
        :is-game-over="isGameOverToast"
        @restart="resetGame"
      />
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
</style>
