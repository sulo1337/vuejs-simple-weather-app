<template>
<div id="app" v-cloak>
    <h1>Weather</h1>
    <div v-if="errorStr">
        Sorry, but the following error has occurred: {{ errorStr }}
    </div>
    <div>
        Your Coordinates: {{ latitude.toFixed(5) }}, {{ longitude.toFixed(5) }}
        <br />
        Updated: {{ updatedAt }}
        <br />
        <br />
        <div v-if="getWeather()">
            <b-jumbotron id="jumbo">
                <template v-slot:header>
                    <img :src="iconUrl" />
                    <br />
                    {{ currentTemp }}°
                </template>
                <template v-slot:lead> Feels like: {{ feelsLike }}° </template>
                <p>{{ this.weather.description }}</p>
                <b-row>
                    <b-col>
                        <img src="https://img.icons8.com/color/50/000000/hot.png" />{{
                maxTemp
              }}°</b-col>
                    <b-col>
                        <img src="https://img.icons8.com/color/50/000000/cold.png" />{{
                minTemp
              }}°
                    </b-col>
                </b-row>
                <b-row>
                    <b-col><img src="https://img.icons8.com/plasticine/100/000000/sunrise.png" class="sun" />
                        <br />{{ this.sunriseUTC }}</b-col>
                    <b-col><img src="https://img.icons8.com/plasticine/100/000000/sunset.png" class="sun" />
                        <br />{{ this.sunsetUTC }}</b-col>
                </b-row>
                <div v-if="unit">
                    <b-button disabled pill variant="light">°C</b-button>
                    <b-button pill variant="light" v-on:click="toggleUnit()">°F</b-button>
                </div>
                <div v-else>
                    <b-button pill variant="light" v-on:click="toggleUnit()">°C</b-button>
                    <b-button disabled pill variant="light">°F</b-button>
                </div>
            </b-jumbotron>
        </div>
    </div>
</div>
</template>

<script>
import axios from "axios";
export default {
    name: "app",
    data() {
        return {
            ipAddr: null,
            updatedAt: null,
            longitude: null,
            latitude: null,
            location: null,
            errorStr: null,
            weatherInfo: null,
            apiurl: null,
            apikey: "584ec8708561cc17a1ca8cfc484e65ee",
            weatherFetched: false,
            weather: null,
            currentTemp: null,
            sunrise: null,
            sunriseUTC: null,
            sunset: null,
            sunsetUTC: null,
            wind: null,
            winddir: null,
            humidity: null,
            feelsLike: null,
            iconUrl: null,
            weatherText: null,
            unit: true,
            maxTemp: null,
            minTemp: 0
        };
    },
    created() {
        fetch("https://api.ipify.org?format=json")
            .then(x => {
                return x.json();
            })
            .then(({
                ip
            }) => {
                this.ipAddr = ip;
                console.log(ip);
                var key = "900dbabca264cc47bb4012d58476d92a";
                var url =
                    "http://api.ipstack.com/" + this.ipAddr + "?access_key=" + key;
                fetch(url)
                    .then(response => {
                        return response.json();
                    })
                    .then(data => {
                        this.latitude = data.latitude;
                        this.longitude = data.longitude;
                    });
            });
    },
    methods: {
        getWeather: function () {
            if (!this.weatherFetched) {
                this.apiurl =
                    "https://api.openweathermap.org/data/2.5/weather?lat=" +
                    this.latitude +
                    "&lon=" +
                    this.longitude +
                    "&appid=" +
                    this.apikey;
                fetch(this.apiurl)
                    .then(response => {
                        return response.json();
                    })
                    .then(data => {
                        this.weatherInfo = data;
                        this.currentTemp = (data.main.temp - 273).toFixed(1);
                        this.feelsLike = (data.main.feels_like - 273).toFixed(1);
                        this.sunrise = data.sys.sunrise;
                        this.sunset = data.sys.sunset;
                        this.wind = data.wind.speed;
                        this.winddir = data.wind.deg;
                        this.humidity = data.main.humidity;
                        this.weather = data.weather[0];
                        this.iconUrl =
                            "http://openweathermap.org/img/wn/" +
                            data.weather[0].icon +
                            "@2x.png";
                        this.maxTemp = (data.main.temp_max - 273).toFixed(1);
                        this.minTemp = (data.main.temp_min - 273).toFixed(1);
                        var currentdate = new Date();
                        this.updatedAt =
                            currentdate.getDate() +
                            "/" +
                            (currentdate.getMonth() + 1) +
                            "/" +
                            currentdate.getFullYear() +
                            " @ " +
                            currentdate.getHours() +
                            ":" +
                            currentdate.getMinutes() +
                            ":" +
                            currentdate.getSeconds();
                    });
                this.weatherFetched = true;
            }
            this.postProcess();
            return true;
        },
        postProcess: function () {
            String.prototype.capitalize = function () {
                return this.replace(/(?:^|\s)\S/g, function (a) {
                    return a.toUpperCase();
                });
            };
            this.weather.description = this.weather.description.capitalize();
            var dateObj = new Date((this.sunrise + 5 * 60 * 60) * 1000);
            var utcString = dateObj.toUTCString();
            this.sunriseUTC = utcString.slice(-11, -7) + " AM";
            dateObj = new Date((this.sunset + 7 * 60 * 60) * 1000);
            utcString = dateObj.toUTCString();
            this.sunsetUTC = utcString.slice(-11, -7) + " PM";
        },
        toggleUnit: function () {
            if (this.unit) {
                this.currentTemp = this.toFahrenheit(this.currentTemp);
                this.feelsLike = this.toFahrenheit(this.feelsLike);
                this.maxTemp = this.toFahrenheit(this.maxTemp);
                this.minTemp = this.toFahrenheit(this.minTemp);
            } else {
                this.currentTemp = this.toCelsius(this.currentTemp);
                this.feelsLike = this.toCelsius(this.feelsLike);
                this.maxTemp = this.toCelsius(this.maxTemp);
                this.minTemp = this.toCelsius(this.minTemp);
            }
            this.unit = !this.unit;
        },
        toFahrenheit: function (num) {
            return (num * (9 / 5) + 32).toFixed(1);
        },
        toCelsius: function (num) {
            return ((num - 32) * (5 / 9)).toFixed(1);
        }
    }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Baloo+Paaji+2&display=swap");

body {
    background-color: #222831;
    color: #ffffff;
}

#app {
    font-family: "Baloo Paaji 2";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    margin-top: 60px;
}

h1,
h2 {
    font-weight: normal;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    display: inline-block;
    margin: 0 10px;
}

a {
    color: #42b983;
}

#jumbo {
    width: 30%;
    margin: auto;
    padding-top: 0;
    padding-bottom: 10px;
    background: #077b8a;
    border-radius: 50px;
    box-shadow: 5px 10px 18px #111111;
    text-shadow: 2px 2px 5px #000000;
}
</style>
