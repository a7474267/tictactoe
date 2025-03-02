<script setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps({
  keyboard: {
    type: Array,
    required: true,
  },
  getKeyStatus: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["keyInput"]);

const handleKeyInput = (key) => {
  emit("keyInput", key);
};
</script>

<template>
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
</template>

<style scoped>
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
</style>
