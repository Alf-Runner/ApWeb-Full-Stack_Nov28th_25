<template>
  <div id="app">
    <PaymentSystem />
  </div>
</template>

<script>
import PaymentSystem from './components/PaymentSystem.vue'

export default {
  name: 'App',
  components: {
    PaymentSystem
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f7fafc;
  color: #2d3748;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
}
</style>
