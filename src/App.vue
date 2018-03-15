<template>
  <div id="app">
    <!-- <HelloWorld/> -->
    <FlowChart/>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import FlowChart from "./components/FlowChart.vue";

export default {
  name: "app",
  components: {
    HelloWorld,
    FlowChart
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
