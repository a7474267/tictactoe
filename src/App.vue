<script setup>
import Wordle from "./components/TicTacToe.vue";
</script>

<template>
  <div class="container">
    <Wordle />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  background-color: #121213;
  color: #ffffff;
}

body {
  background-color: #121213;
  min-height: 100vh;
  margin: 0;
  padding: 0;
}

#app {
  display: flex;
  justify-content: center;
  min-height: 100vh;
  background-color: #121213;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
  width: 100%;
}
</style>
