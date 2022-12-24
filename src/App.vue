<template>
  <Trades />
</template>

<script>
import Trades from "./components/Trades.vue";
export default {
  name: "App",
  components: {
    Trades: Trades,
  },
};
</script>

<style>
body {
  background-color: #999;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  font-size: 9px;
}
@media screen and (min-width: 800px) {
  #app {
    font-size: 16px;
  }
}
</style>
