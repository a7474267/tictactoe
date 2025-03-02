<script setup>
import Wordle from "./components/TicTacToe.vue";
</script>

<template>
  <div class="container">
    <Wordle />
  </div>
</template>

<style>
:root {
  background-color: #121213;
  color: #ffffff;
}

#app {
  display: flex;
  justify-content: center;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
  width: 100%;
}

h1 {
  color: #ffffff;
  margin-bottom: 2rem;
}
</style>
