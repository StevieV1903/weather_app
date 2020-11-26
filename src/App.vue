<template>
  <div id="app">
     <h1 class="title">Worldwide Weather</h1>
    <h3 className="header-subtext">Find out the current weather conditions in any country around the world!</h3>

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
  color: black;
  clear: both;
  background-color:#c1c8e4;
  padding-top: 25px;
}
body{
  margin: 0px !important;

}
#app > h1 {
  margin: 0;
}
#app > h3 {
  margin: 0;
  padding: 0 20px
}

</style>
