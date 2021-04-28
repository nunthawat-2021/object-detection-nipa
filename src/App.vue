<template>
  <div id="app">
   <div>
  <ImageDetect msg="Welcome to Your Vue.js App"/>
   </div>
   
  </div>
</template>

<script>
import ImageDetect from './components/ImageDetect.vue'

export default {
  name: 'App',
  components: {
    ImageDetect
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
