<template>

    <div class="app-container">

        <div class="hidden-dropdown" v-if="this.searchLocation">
                <select v-if="this.searchLocation.results.length > 1" v-model="selectedLocation" v-on:change="sublocationWeatherFetch">
                    <option value="Other locations for this search:" disabled hidden>Other locations for this search...</option>
                    <option v-for="(location, index) in this.searchLocation.results" v-bind:value="location" v-bind:key="index">{{ location.address.freeformAddress }}</option>
                </select>
                </div>
        
        <div class="title-container">
            <h2 v-if="this.searchLocation != null" class="location-detail"> {{ this.searchLocationName }}</h2>
                
            <h2 v-if="this.currentLocationDetails != null" class="location-title"> 
                {{ this.currentLocationDetails.addresses[0].address.municipality }}  
                ({{ this.currentLocationDetails.addresses[0].address.postalCode }})</h2>
            <h3 v-if="this.currentLocationDetails != null" class="location-detail">
                {{ this.currentLocationDetails.addresses[0].address.countrySecondarySubdivision}},
                {{ this.currentLocationDetails.addresses[0].address.country }}

            </h3>
        </div>
    <div class="weather-wrapper">
        <div class="weather-container">
            <h2 class="current-weather">Current Weather:</h2>
            <h2 v-if="this.currentLocationWeather != null" class="weather"> {{ this.currentLocationWeather.current.weather[0].description }}</h2>
                <div class="image-temp-grid">
                    <img :src=this.weatherIcon class="image-fit">
                
                    <h2 v-if="this.currentLocationWeather != null" class="temperature">{{ Math.round(this.currentLocationWeather.current.temp)  }}°C</h2>
                </div>
            
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed"> wind-speed: {{ this.getWindMilesPerHour(this.currentLocationWeather.current.wind_speed)  }}mph</h3>
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed"> cloud cover: {{ (this.currentLocationWeather.current.clouds)  }}%</h3>
                <div class="other-weather-details-container">
                <h3 v-if="this.currentLocationWeather != null" class="sun-rise"> 
                    <img src="../assets/sunrise.png" class="grid-image">
                {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunrise) }}</h3>
                <h3 v-if="this.currentLocationWeather != null" class="sun-rise">
                    <img src="../assets/sunset.png" class="grid-image"> {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunset) }}</h3>
                </div>
        </div>
        <div class="dates-wrapper">
        <h2 class="daily-forecast-title">Click on a date below for daily forecast:</h2>
        <ul v-if="this.currentLocationWeather != null" :refresh='refresh' class="dates-list">
            <li v-bind:class="{dateClicked: isVisible[index]}" v-for="(item, index) in this.currentLocationWeather.daily" :key="item.dt" class="dropdown-weather-text">
                <button class="date-button" v-on:click="toggleIsVisible(index)"><a href="#weather-description"> {{ convertDateFromTimeStamp(item.dt) }}</a></button>
    
                <h2 v-if="isVisible[index] === true" id="weather-description"> {{ item.weather[0].description }} ({{ Math.round(item.temp.day) }}°C)</h2>
                <h2 v-if="isVisible[index] === true"> temp am: {{ Math.round(item.temp.morn) }}°C</h2>
                <h2 v-if="isVisible[index] === true"> temp pm: {{ Math.round(item.temp.night) }}°C</h2>
                <h2 v-if="isVisible[index] === true"> wind: {{ Math.round( item.wind_speed * 2.237 ) }}mph</h2>
                <h2 v-if="isVisible[index] === true"> clouds: {{ item.clouds }}%</h2>
            
            </li>
        </ul>
        </div>
    </div>
</div> 
</template>

<script>

import apiKey from "../apikey.js"
import { eventBus } from "../main.js"
const tzlookup = require("tz-lookup")

export default {

    data() {
        return {
            currentCoordinates: null,
            currentLocationWeather: null,
            currentLocationDetails: null,
            weatherIcon: null,
            isVisible: [false, false, false, false, false, false, false, false],
            refresh: false,
            searchLocationName: null,
            searchLocation: null,
            selectedLocation: "Other locations for this search:"
        }
    },

    methods: {
        getCoordinates() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition((position) => {
                     fetch('https://api.openweathermap.org/data/2.5/onecall?lat=' + position.coords.latitude + '&lon=' + position.coords.longitude + '&units=metric&appid=' + apiKey.weatherKey )
                    .then(res => res.json())
                    .then(results => this.currentLocationWeather = results)
                    .then((results) => this.weatherIcon = "http://openweathermap.org/img/wn/" +  results.current.weather[0].icon + "@2x.png")
                    .then(() => this.getLocationName(position)) 

                    this.currentCoordinates = position.coords
                })
            } else {
                console.log("not working");
            }
        },

        getLocationName(position) {
            fetch("https://api.tomtom.com/search/2/reverseGeocode/" + position.coords.latitude + "%2C" + position.coords.longitude + ".json?key=" + apiKey.tomTomKey)
            .then(res => res.json())
            .then(results => this.currentLocationDetails = results)
        },

        sublocationWeatherFetch() {
            fetch('https://api.openweathermap.org/data/2.5/onecall?lat=' + this.selectedLocation.position.lat + '&lon=' + this.selectedLocation.position.lon + '&units=metric&appid=' + apiKey.weatherKey )
          .then(res => res.json())
          .then(results => this.currentLocationWeather = results)
          .then((results) => this.weatherIcon = "http://openweathermap.org/img/wn/" +  results.current.weather[0].icon + "@2x.png")
          .then(() => this.searchLocationName = this.selectedLocation.address.freeformAddress)
        },

        convertTimeFromTimeStamp(path) {
            const date = new Date(path * 1000)
              let timeZoneString = ""
            if(this.selectedLocation.position){
                timeZoneString = tzlookup(this.selectedLocation.position.lat, this.selectedLocation.position.lon)
            } else if(this.searchLocation) { 
                timeZoneString = tzlookup(this.searchLocation.results[0].position.lat, this.searchLocation.results[0].position.lon)
            } else {
                timeZoneString = tzlookup(this.currentCoordinates.latitude, this.currentCoordinates.longitude)
            }
            const time = date.toLocaleTimeString("en-GB", {hour: "2-digit", minute: "2-digit", timeZone: timeZoneString})

            return time
        },
        convertDateFromTimeStamp(path) {
            const date = new Date(path * 1000)
            const formattedDate = date.toLocaleDateString()
            return formattedDate
        },
        toggleIsVisible(index) {
            const tempIsVisible = this.isVisible

            if (tempIsVisible[index] === true){
                tempIsVisible[index] = false
            } else {
                tempIsVisible[index] = true
            }

            this.isVisible = tempIsVisible
            this.refreshPage()
        },

        refreshPage() {
            this.refresh = !this.refresh
        },

        getWindMilesPerHour(metresPerSecondFigure) {
            const milesPerHourFigure = metresPerSecondFigure * 2.237
            return Math.round(milesPerHourFigure)
        }
            

    },

    mounted() {
        this.getCoordinates()
        eventBus.$on("searched-weather-data", ({searchLocationWeather, searchLocationResults, weatherIcon}) => {
            this.currentLocationWeather = searchLocationWeather
            this.currentLocationDetails = null
            this.searchLocation = searchLocationResults
            this.searchLocationName = searchLocationResults.results[0].address.freeformAddress
            this.weatherIcon = weatherIcon
            this.selectedLocation = "Other locations for this search:"
        })
    }
}




</script>

<style scoped>
@import url(
    'https://fonts.googleapis.com/css2?family=Bubbler+One&family=Prompt:wght@100;400;700&family=Ubuntu+Mono:wght@400;700&display=swap'
    );
@import url(
    'https://fonts.googleapis.com/css2?family=Roboto+Condensed&display=swap'
    );
@import url(
    'https://fonts.googleapis.com/css2?family=Amatic+SC:wght@400;700&family=Grandstander:wght@400;700&family=Pridi:wght@400;700&display=swap'
    );

/* font-family: 'Amatic SC', cursive;
font-family: 'Grandstander', cursive;
font-family: 'Pridi', serif; */

.app-container {
    background-color: #f6f7ff;
    min-height: 100vh;
    margin: 0;
    padding: 20px;
}
.title-container {
    border: solid 4px #8860d0;
    padding: 15px;
    border-radius: 20px;
    max-width: 400px;
    margin-left: auto;
    margin-right: auto; 
    background-color: #b9e9fb;
}
.location-title {
    font-family: 'Pridi', serif;
    font-size: 30px;
    margin: 0px;
    padding: 5px 5px;
    color: #6b00a8;
}
.location-detail {
    font-family: 'Pridi', serif;
    font-size: 24px;
    padding: 5px 5px;
    color: #6b00a8;
    margin: 0;
}
.weather-wrapper {
    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    margin-right: auto;
    margin-left: auto;
    margin-bottom: 15px;
}
.dates-wrapper {
    border: solid 4px #8860d0;
    border-radius: 20px;
    max-width: 350px;
    margin-top: 20px;
    /* margin-left: auto;
    margin-right: auto;  */
    background-color: #b9e9fb; 
    padding: 10px 20px;
    height: 565px;
    overflow: scroll;
}
.weather-container {
    border: solid 4px #8860d0;
    border-radius: 20px;
    max-width: 350px;
    margin-top: 20px;
    /* margin-left: auto;
    margin-right: auto;  */
    background-color: #b9e9fb; 
    padding: 10px 20px;
    min-height: 560px;
    justify-self: right;
    margin-right: 10px;

}
.current-weather{
    font-size: 26px;
    font-family: 'Grandstander', cursive;
    margin: 0px;
    margin-top: 10px;
    color: #6b00a8;
}
.weather{
    font-size: 50px;
    font-family: 'Amatic SC', cursive;
    font-weight: bold;
    margin: 0px;
    color: black;
    text-transform: capitalize;
}
.image-temp-grid {
    display: grid;
    grid-template-columns: 50% 50%;
    align-items: center;
}
.image-fit{
    height: 180px;
}
.temperature {
    font-family: 'Amatic SC', cursive;
    color: #5f5f5f;
    font-size: 50px;
    padding: 25px 0;
    background-color: #ffffff;
    border: solid 5px #8860d0;
    border-radius: 20px;
    margin: 0 10px;
    box-shadow: 5px 5px 5px grey;
}
.sun-rise { 
    font-family: 'Amatic SC', cursive;
    font-weight: bold;
    font-size: 40px;
    margin: 0px;
    color: black;
}
select {
  display: block;
  font-family: 'Bubbler One', sans-serif;
  font-size: 20px;
  color: #444;
  padding: 6px;
  width: 70%;
  max-width: 100%; /* useful when width is set to anything other than 100% */
  box-sizing: border-box;
  margin: 0;
  margin-left: auto;
  margin-right: auto;
  border: 4px solid #8860d0;
  border-radius: 8px;
  background-color: #fff;
}
@media screen and (max-width: 450px) {
  .image-fit {
      height: 140px;
  }
  .temperature {
    font-size: 45px;
    margin: 10px;
}
}

.wind-speed {
    font-size: 30px;
    font-family: 'Amatic SC', cursive;
    margin: 0px;
    color: black;
}
.other-weather-details-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    margin-left: auto;
    margin-right: auto; 
    background-color: #b9e9fb;
    
}
.other-weather-details-container h2{
    font-family: 'Ubuntu Mono', monospace;
    margin: 0;
    
}
.hidden-dropdown {
    padding-bottom: 15px;
}
.dates-list{
    list-style-type: none;
    margin: 0px;
    padding: 0px;
}

@media screen and (max-width: 950px) {
  /* .dates-list {
    display: block;
  } */
}

.daily-forecast-title {
    font-family: 'Grandstander', cursive;
    font-size: 26px;
    margin: 0px;
    color: #6b00a8;
    margin-top: 10px;
    margin-bottom: 15px;
}
.date-button {
    margin-top: 5px;
    background-color:#8860d0;
    border: solid 1px #5800fb;
    border-radius: 10px;
    padding: 5px 10px;
    cursor: pointer;
}
.date-button > a {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 20px;
    text-decoration: none;
    background-color:#8860d0;
    color: white;
    
}
.dropdown-weather-text {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 12px;
    color: black;
    padding: 5px;
    margin: 0px;
}
.dateClicked {
    border: solid 4px #8860d0;
    background-color: #b9e9fb;
    border-radius: 20px;
    max-width: 200px; 
    margin-left: auto;
    margin-right: auto;
    padding: 0px 15px;
    margin-top: 15px;

}





.grid-image {
    height: 125px;
    /* object-fit: fill; */
}

@media screen and (max-width: 365px) {
.grid-image {
    height: 90px;
}
}

@media screen and (max-width: 325px) {
.temperature {
    margin: 5px;

}
}


</style>