<template>
  <div id="app">
    <Home />
  </div>
</template>

<script>

import Home from './components/Home';

export default {
  name: 'App',
  components: {
    Home
  }
}
</script>

<style>
  #app{
    padding: 0 80px; 
  }
</style>
