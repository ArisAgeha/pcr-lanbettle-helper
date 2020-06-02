<template>
  <div id="app">
    <Analyzer />
  </div>
</template>

<script>
import Analyzer from "./components/Analyzer.vue";

export default {
  name: "App",
  components: {
    Analyzer
  }
};
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
}

*::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
*::-webkit-scrollbar-thumb {
  border-radius: 6px;
  background: rgba(64, 158, 255, 1);
}
*::-webkit-scrollbar-track {
  border-radius: 0;
  background: rgba(116, 116, 117, 0);
}
</style>
