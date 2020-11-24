<template>
  <div id="app">
     <h1 class="title">Worldwide Weather</h1>
    <h3>Welcome to the only weather app you'll ever need!
    Find out the current weather consitions in any country around the world!</h3>

    <current-location-weather />
    
  </div>
</template>

<script>

import CurrentLocationWeather from './components/CurrentLocationWeather.vue'
export default {
  name: 'App',
  components: {
    "current-location-weather": CurrentLocationWeather
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
  clear: both;
  background-color:#f1f1f1;
  padding-top: 10px;
}
body{
  margin: 0px !important;
}

</style>
