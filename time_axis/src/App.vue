<template>
  <div id="app">
    <TimeAxis></TimeAxis>
  </div>
</template>

<script>
import TimeAxis from "./components/TimeAxis.vue";

export default {
  components: {
    TimeAxis,
  },
};
</script>

<style>
#app {
  position: absolute;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}
</style>
