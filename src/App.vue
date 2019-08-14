<template>
  <div id="app">
    <Test/>
  </div>
</template>

<script>
import Test from './components/Test.vue'

export default {
  name: 'app',
  components: {
    Test
  }
}
</script>

<style>
* {
  padding: 0;
  margin: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
