<template>
  <div id="app">
    <Editor/>
  </div>
</template>

<script>
import Editor from './components/Editor'

export default {
  name: 'App',
  components: {
    Editor
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 100%;
  justify-content: center;
  display: flex;
  padding: 50px 0;
}
</style>
