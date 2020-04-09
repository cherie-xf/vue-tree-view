<template>
  <div id="app">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <vu-tree></vu-tree>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import vuTree from './components/tree.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld,
    vuTree,
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
