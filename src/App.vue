<template>
  <div id="app">
<!--    <BasicModelVisualization/>-->
    <CarModel />
  </div>
</template>

<script>
// import BasicModelVisualization from './components/BasicModelVisualization.vue'

import CarModel from "@/components/CarModel/CarModel";
export default {
  name: 'App',
  components: {
    CarModel
    // BasicModelVisualization
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
