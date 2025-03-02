<script setup>
const props = defineProps({
  show: Boolean,
  message: String,
  isGameOver: Boolean,
});

const emit = defineEmits(["restart"]);

const handleRestart = () => {
  emit("restart");
};
</script>

<template>
  <div v-if="show" class="toast" :class="{ 'game-over': isGameOver }">
    {{ message }}
    <button v-if="isGameOver" class="restart-button" @click="handleRestart">
      重新開始
    </button>
  </div>
</template>

<style scoped>
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
