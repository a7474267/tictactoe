<script setup>
import { ref, watch, onUnmounted } from "vue";

const props = defineProps({
  isActive: Boolean,
});

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
let interval;
const startFireworks = () => {
  if (!props.isActive) return;

  interval = setInterval(() => {
    if (!props.isActive) {
      clearInterval(interval);
      fireworks.value = [];
      return;
    }
    for (let i = 0; i < 3; i++) {
      setTimeout(() => createFirework(), i * 50);
    }
  }, 200);
};

// 監聽 isActive 變化
watch(
  () => props.isActive,
  (newValue) => {
    if (newValue) {
      startFireworks();
    } else {
      clearInterval(interval);
      fireworks.value = [];
    }
  }
);

// 組件卸載時清理
onUnmounted(() => {
  clearInterval(interval);
  fireworks.value = [];
});
</script>

<template>
  <div v-if="isActive" class="fireworks-container">
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
</template>

<style scoped>
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
</style>
