<template>
    <div class="form-container">
        <form @submit.prevent >
                <label class="form-label">Search for a Country:</label>
                <select v-model="selectedCountry" required> 
                <option value="null" disabled hidden>Select a Country from this list...</option>
                <option v-for="(country, index) in this.countriesData" v-bind:value="country" v-bind:key="index">{{ country.name }}</option>
                </select>
                <br/>

                <label class="form-label">Location Name: </label>
                <input class="form-input" v-model="searchTerm" placeholder="Type a location name here..."/>
                <br/>
                <button v-on:click="submit" class="form-button">Submit</button>
        </form>
    </div>
</template>

<script>

import apiKey from "../apikey.js"
import { eventBus } from '../main.js'
export default {
  name: 'SearchLocation',
  data() {
      return {
          searchTerm: "",
          searchLocationResults: null,
          searchLocationWeather: null,
          weatherIcon: null,
          countriesData: null,
          selectedCountry: null
      }
  },
  methods: {
      submit() {
            fetch("https://api.tomtom.com/search/2/structuredGeocode.json?countryCode=" + this.selectedCountry.alpha3Code + "&municipalitySubdivision=" + this.searchTerm + "&key=" + apiKey.tomTomKey )
            .then(res => res.json())
            .then(results => {
                this.searchLocationResults = results
            })
            .then(() => {
                if(this.searchLocationResults.results.length === 0) {
                    alert("Please enter a valid location.")
                } else {
                    this.conditionalFetch() 
                }
            })
          
      },
      conditionalFetch(){
          fetch('https://api.openweathermap.org/data/2.5/onecall?lat=' + this.searchLocationResults.results[0].position.lat + '&lon=' + this.searchLocationResults.results[0].position.lon + '&units=metric&appid=' + apiKey.weatherKey )
          .then(res => res.json())
          .then(results => this.searchLocationWeather = results)
          .then((results) => this.weatherIcon = "http://openweathermap.org/img/wn/" +  results.current.weather[0].icon + "@2x.png")
          .then(() => eventBus.$emit("searched-weather-data", {searchLocationWeather: this.searchLocationWeather, searchLocationResults: this.searchLocationResults, weatherIcon: this.weatherIcon}))
      }

  },
  mounted() {
      fetch("https://restcountries.eu/rest/v2/all")
        .then(res => res.json())
        .then(results => this.countriesData = results)
  }

}
</script>

<style scoped>
@import url(
    'https://fonts.googleapis.com/css2?family=Amatic+SC:wght@400;700&family=Grandstander:wght@400;700&family=Pridi:wght@400;700&display=swap'
    );

.form-container {
    padding: 20px 100px; 
    box-sizing: border-box;
}
form {
  border-radius: 5px;
  background-color: #f2f2f2;
  padding: 20px;
  border: 4px solid #8860d0;
  border-radius: 8px;
  text-align: left;
}

.form-label {
    font-family: 'Pridi', serif;
    font-size: 18px;
    font-weight: bold;
    padding: 5px 0px;
    color: #6b00a8;
    display: inline-block;
    /* display: block; */
}

.form-button {
    margin-top: 15px;
    font-family: 'Grandstander', cursive;
    font-size: 18px;
    background-color:#6b00a8;
    color: white;
    border: solid 1px white;
    border-radius: 10px;
    padding: 10px 20px;
    cursor: pointer;
}
.form-button:hover {
    margin-top: 15px;
    color:#6b00a8;
    background-color: white;
    border: solid 1px #8860d0;
}
select, input {
  font-size: 18px;
  font-family: 'Bubbler One', sans-serif;
  font-weight: bold;
  color: #444;
  padding: 6px;
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
  margin: 0;
  margin-left: auto;
  margin-right: auto;
  border: 4px solid #8860d0;
  border-radius: 8px;
  background-color: #fff;
}


/* RESPONSIVE RULES:  */
@media screen and (max-width: 600px) {
  .form-container {
        padding: 20px; 
        box-sizing: border-box;
}

}
</style>