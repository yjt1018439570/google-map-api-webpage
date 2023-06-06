<template>
  <div id="app">
    <Map />
    <current />
  </div>
</template>

<script>
import Map from "./components/Map.vue";
import current from "./components/current.vue"

export default {
  components: {
    Map,
    current
  }
};

</script>

<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
}
</style>