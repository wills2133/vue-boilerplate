<template>
  <div id="app">
    <phoneLogin />
  </div>
</template>

<script>
import phoneLogin from './components/phoneLogin.vue'

export default {
  name: 'app',
  components: {
    phoneLogin
  }
}
</script>

<style>
  @import '~flag-icon-css/css/flag-icon.min.css';
</style>
