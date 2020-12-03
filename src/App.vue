<template>
  <div id="app">
    <div class="title-wrapper">
      <img src="./assets/www-logo.png" class="title-image"> 
      <h1 class="title">Worldwide Weather</h1>
    </div>
    
    <h3 class="header-subtext">Find out the current weather conditions in any location around the world!</h3>
   

    <search-weather-location />
    <current-location-weather />
    
    
  </div>
</template>

<script>

import CurrentLocationWeather from './components/CurrentLocationWeather.vue'
import SearchWeatherLocation from './components/SearchWeatherLocation.vue'

export default {
  name: 'App',
  components: {
    "current-location-weather" : CurrentLocationWeather,
    "search-weather-location" : SearchWeatherLocation  
  }

}
</script>

<style>
@import url(
    'https://fonts.googleapis.com/css2?family=Amatic+SC:wght@400;700&family=Grandstander:wght@400;700&family=Pridi:wght@400;700&display=swap'
    );

#app {
  text-align: center;
  color: black;
  clear: both;
  background-color:#f6f7ff;
  padding-top: 25px;
}
body{
  margin: 0px !important;

}
.title {
  margin: 0;
  font-family: 'Pridi', serif;
  color: #6b00a8;
  font-size: 32px;
  
}
#app > h3 {
  margin: 0;
  padding: 0 20px;
  font-family: 'Grandstander', cursive;
}
.title-wrapper {
  display: grid;
  grid-template-columns: 1fr 2fr;
  align-items: center;
  width: fit-content;
  margin-right: auto;
  margin-left: auto;
  margin-bottom: 15px;
}
.title-image {
  width: 80px;
  justify-self: right;
  margin-right: 10px;
  border: solid 4px #8860d0; 
  border-radius: 20px;
}

/* RESPONSIVE RULES:  */

@media screen and (max-width: 450px) {
  .title-wrapper {
    grid-template-columns: 20% 80%;
}
  .title {
    font-size: 24px;
}
  .title-image {
    width: 50px;
  }
}

</style>
