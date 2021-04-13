<template>
  <div id="app">
    <vu-tree></vu-tree>
  </div>
</template>

<script>
import vuTree from './components/tree.vue'

export default {
  name: 'App',
  components: {
    vuTree,
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 60px;
}
</style>
