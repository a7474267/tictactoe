<script setup>
const props = defineProps({
  board: Array,
  currentAttempt: Number,
  getLetterStatus: Function,
});
</script>

<template>
  <div class="board">
    <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
      <div
        v-for="(letter, colIndex) in row"
        :key="colIndex"
        class="cell"
        :class="[
          getLetterStatus(letter, colIndex, rowIndex),
          {
            active: rowIndex === currentAttempt,
            filled: letter !== '',
          },
        ]"
      >
        {{ letter }}
      </div>
    </div>
  </div>
</template>

<style scoped>
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
</style>
