<template>

    <div class="app-container">
        <div class="title-container">
            <h2 v-if="this.currentLocationDetails != null" class="location-title"> 
                {{ this.currentLocationDetails.addresses[0].address.municipality }}  
                ({{ this.currentLocationDetails.addresses[0].address.postalCode }})</h2>
            <h3 v-if="this.currentLocationDetails != null" class="location-detail">
                {{ this.currentLocationDetails.addresses[0].address.countrySecondarySubdivision}},
                {{ this.currentLocationDetails.addresses[0].address.country }}

            </h3>
        </div>
    
        <div class="weather-container">
            <h2 v-if="this.currentLocationWeather != null" class="current-weather">current weather {{ this.currentLocationWeather.current.weather[0].description }}</h2>
            <div class="image-temp-grid">
            <img :src=this.weatherIcon class="image-fit">
        
            <h2 v-if="this.currentLocationWeather != null" class="temperature">{{ Math.round(this.currentLocationWeather.current.temp)  }}°C</h2>
            </div>
            <!-- <div class="other-weather-details-container"> -->
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed"> windspeed: {{ this.getWindMilesPerHour(this.currentLocationWeather.current.wind_speed)  }}mph</h3>
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed"> cloud cover: {{ (this.currentLocationWeather.current.clouds)  }}%</h3>
            <div class="other-weather-details-container">
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed"> 
                <img src="../assets/sunrise.png" class="grid-image">
             {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunrise) }}</h3>
            <h3 v-if="this.currentLocationWeather != null" class="wind-speed">
                <img src="../assets/sunset.png" class="grid-image"> {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunset) }}</h3>
            </div>
            </div>
        
    <!-- <div class="other-weather-details-container">
        
    </div> -->
        <h2 class="daily-forecast-title">Click on a date below for daily forecast:</h2>
        <ul v-if="this.currentLocationWeather != null" :refresh='refresh' class="dates-list">
            <li v-for="(item, index) in this.currentLocationWeather.daily" :key="item.dt" class="dropdown-weather-text">
                <button class="date-button" v-on:click="toggleIsVisible(index)"><a href="#weather-description"> {{ convertDateFromTimeStamp(item.dt) }}</a></button>
    
                <h2 v-if="isVisible[index] === true" id="weather-description"> {{ item.weather[0].description }} ({{ Math.round(item.temp.day) }}°C)</h2>
                <h2 v-if="isVisible[index] === true"> temp am: {{ Math.round(item.temp.morn) }}°C</h2>
                <h2 v-if="isVisible[index] === true"> temp pm: {{ Math.round(item.temp.night) }}°C</h2>
            
            </li>
        </ul>
        </div> 
</template>

<script>

import apiKey from "../apikey.js"

export default {

    data() {
        return {
            currentCoordinates: null,
            currentLocationWeather: null,
            currentLocationDetails: null,
            weatherIcon: null,
            isVisible: [false, false, false, false, false, false, false, false],
            refresh: false

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

        convertTimeFromTimeStamp(path) {
            const date = new Date(path * 1000)
            const time = date.toLocaleTimeString("en-US", {hour: "2-digit", minute: "2-digit"})
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
    }
}




</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Bubbler+One&family=Prompt:wght@100;400;700&family=Ubuntu+Mono:wght@400;700&display=swap');

.app-container {
    background-color:#f1f1f1;
    min-height: 100vh;
    margin: 0;
    padding: 20px;
}
.title-container {
    border: solid 4px #5f5f5f;
    padding: 15px;
    border-radius: 20px;
    max-width: 400px;
    margin-left: auto;
    margin-right: auto; 
    background-color: #aefadd;
}
.location-title {
    font-size: 35px;
    margin: 0px;
    padding: 5px 10px;
    color: #5f5f5f;
}
.location-detail {
    font-size: 26px;
    padding: 5px 10px;
    color: #5f5f5f;
    margin: 0;
}
.weather-container {
    border: solid 4px #5f5f5f;
    border-radius: 20px;
    max-width: 350px;
    margin-top: 20px;
    margin-left: auto;
    margin-right: auto; 
    background-color: #00ffff; 
    padding: 10px 20px;
}
.current-weather{
    font-size: 30px;
    margin: 0px;
    padding: 20px 10px 5px 10px;
    color: #5f5f5f;
    text-transform: capitalize;
}
.image-temp-grid {
    display: grid;
    grid-template-columns: 50% 50%;
    align-items: center;
    margin-right: 20px;
}
.image-fit{
    height: 180px;
}
.temperature {
    font-family: 'Prompt', sans-serif;
    color: #5f5f5f;
    /* text-shadow: 2px 2px 5px rgb(255, 173, 173); */
    font-size: 50px;
    padding: 20px;
    background-color: #ffffff;
    border: solid 1px #5f5f5f;
    border-radius: 50%;
    margin: 20px;
    box-shadow: 5px 5px 5px grey;
}
.wind-speed {
    font-size: 28px;
    margin: 0px;
    padding: 0px 5px 5px 5px;
    color: #5f5f5f;
}
.other-weather-details-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    font-family: 'Ubuntu Mono', monospace;
    border: solid 1px black;
    border-radius: 20px;
    /* width: 350px; */
    margin-left: auto;
    margin-right: auto; 
    padding: 10px 0px;
    margin-top: 20px;
    background-color: white;
    
}
.other-weather-details-container h2{
    font-family: 'Ubuntu Mono', monospace;
    margin: 0;
    
}
.dates-list{
    list-style-type: none;
    margin: 0px;
    padding: 0px;
    display: flex;
    flex: wrap;
    justify-content: center;
}

.daily-forecast-title {
    font-family: 'Ubuntu Mono', monospace;
}
.date-button {
    padding: 10px 12px;
    margin-top: 0px;
    border: solid 1px black;
    border-radius: 20px;
    background-color: #90fdfd;
    /* box-shadow: 10px 10px 5px grey; */
    
}
.date-button a {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 16px;
    text-decoration: none;
    color: black;
    
}
.dropdown-weather-text {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 12px;
    color: #5f5f5f;
    padding: 5px;
    margin: 0px;
    /* background-color: aqua; */
}
.grid-image {
    height: 150px;
}



</style>