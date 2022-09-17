<template>
  <Map></Map>
</template>

<script>
import Map from './components/Map.vue'

export default {
  name: 'App',
  components: {
    Map
  }
}
</script>

<style>
#map{
  height: 70%;
  width: 100%;
}

html, body, #app {
  width: 100%;
  height: 100%;
  margin: 0;
}
</style>
