<template>
  <SignificantTrades />
</template>

<script>
import SignificantTrades from "./components/SignificantTrades.vue";
export default {
  name: "App",
  components: {
    SignificantTrades:SignificantTrades
  },
};
</script>

<style>
body {
  background-color: #fff;
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

@media (prefers-color-scheme: dark) {
  body {
    background-color: #999;
    color: white;
  }
  button,
  input,
  select {
    background-color: #999;
    color: #ddd;
  }
}
</style>
