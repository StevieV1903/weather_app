<template>
    <div class="app-container">
        <div class="title-container">
            <h2 v-if="this.currentLocationDetails != null" class="location-title">Your current location is; <br/> {{ this.currentLocationDetails.addresses[0].address.municipality }},  {{ this.currentLocationDetails.addresses[0].address.postalCode }}.</h2>
        </div>
    
        <div class="weather-container">
            <h2 v-if="this.currentLocationWeather != null" class="current-weather"> {{ this.currentLocationWeather.current.weather[0].description }}</h2>
            <div class="image-temp-grid">
            <img :src=this.weatherIcon class="image-fit">
        
            <h2 v-if="this.currentLocationWeather != null" class="temperature">{{ Math.round(this.currentLocationWeather.current.temp)  }}°C</h2>
            </div>
        </div>
        
    <div class="other-weather-details-container">
        <h2 v-if="this.currentLocationWeather != null"> Sunrise: {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunrise) }}</h2>
        <h2 v-if="this.currentLocationWeather != null"> Sunset: {{ this.convertTimeFromTimeStamp(this.currentLocationWeather.current.sunset) }}</h2>
        <h2 v-if="this.currentLocationWeather != null"> Wind Speed: {{ this.currentLocationWeather.current.wind_speed  }}m/s</h2>
    </div>
        <h2 class="daily-forecast-title">Click on a date below for daily forecast:</h2>
        <ul v-if="this.currentLocationWeather != null" :refresh='refresh' class="dates-list">
            <li v-for="(item, index) in this.currentLocationWeather.daily" :key="item.dt" class="dropdown-weather-text">
                <button class="date-button" v-on:click="toggleIsVisible(index)"><a href="#weather-description"> {{ convertDateFromTimeStamp(item.dt) }}</a></button>
                <div class="daily-weather-details">
                <h2 v-if="isVisible[index] === true"> morning temp: {{ Math.round(item.temp.morn) }}°C</h2>
                <h2 v-if="isVisible[index] === true"> day temp: {{ Math.round(item.temp.day) }}°C</h2>
                <h2 v-if="isVisible[index] === true"> night temp: {{ Math.round(item.temp.night) }}°C</h2>
                <h2 v-if="isVisible[index] === true" id="weather-description"> weather: {{ item.weather[0].description }}</h2>
                </div>
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
        }
            

    },

    mounted() {
        this.getCoordinates()
    }
}




</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Ubuntu+Mono:wght@400;700&display=swap');

.app-container {
    background-color:#f1f1f1;
    padding-bottom: 10px;
    min-height: 100vh;
}
.title-container {
    border: solid 1px black;
    border-radius: 20px;
    width: fit-content;
    margin-left: auto;
    margin-right: auto;   
}
.location-title {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 28px;
    margin: 10px;
    padding: 20px;
}
.weather-container {
    border: solid 1px black;
    border-radius: 20px;
    width: fit-content;
    margin-top: 30px;
    margin-left: auto;
    margin-right: auto; 
    background-color: #90fdfd; 
}
.current-weather{
    font-family: 'Ubuntu Mono', monospace;
    font-size: 36px;
    margin: 0px;
    padding: 20px 10px 5px 10px;
}
.image-temp-grid {
    display: grid;
    grid-template-columns: 50% 50%;
    align-items: center;
}
.image-fit{
    height: 175px;
    margin-left: auto;
    margin-right: auto; 
}
.temperature {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 60px;
    padding: 20px;
    margin: 0;
}
.other-weather-details-container {
    font-family: 'Ubuntu Mono', monospace;
    border: solid 1px black;
    border-radius: 20px;
    width: 350px;
    margin-left: auto;
    margin-right: auto; 
    padding: 15px;
    margin-top: 20px;
    
}
.dates-list{
    list-style-type: none;
    margin: 0px;
    padding: 0px;
    display: flex;
    justify-content: center;
}

.daily-forecast-title {
    font-family: 'Ubuntu Mono', monospace;
}
.date-button {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 16px;
    padding: 10px 12px;
    margin: 10px;
    border: solid 1px black;
    border-radius: 20px;
    background-color: aqua;
    
}
.dropdown-weather-text {
    font-family: 'Ubuntu Mono', monospace;
    font-size: 12px;
    color: black;
    padding: 10px;
}



</style>