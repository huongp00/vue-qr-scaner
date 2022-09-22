<template>
  <div id="app">
  <h1> QR code </h1>
  <QRcode />
  </div>
</template>

<script>
import QRcode from './components/QRcode.vue'

export default {
  name: 'App',
  components: {
    QRcode,
  },
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
