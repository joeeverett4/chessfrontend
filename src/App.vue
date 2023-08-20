<template>
  <div id="app">
    
    <ChessAi />
  </div>
</template>

<script>
import ChessAi from './components/ChessAi.vue'

export default {
  name: 'App',
  components: {
    ChessAi
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
