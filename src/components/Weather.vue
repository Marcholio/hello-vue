<template>
  <v-container fluid fill-height>
    <v-col>
      <v-row>
        <v-col>
          <v-row>
            <v-col>
              <!-- Main header -->
              <v-row justify="center">
                <h1>Weather {{ date }}</h1>
                <div id="citySelector">
                  <v-select
                    item-text="name"
                    item-value="id"
                    :items="cities"
                    v-model="selectedCity"
                    outlined
                    @change="updateData"
                  />
                </div>
              </v-row>

              <!-- Temperature -->
              <v-row v-if="!loadingWeather">
                <v-col align-self="center"
                  ><v-row>
                    <v-col md="8">
                      <span id="main-temp">
                        {{ weatherData.main.temp > 0 ? "+" : ""
                        }}{{ weatherData.main.temp }} &#8451;
                      </span></v-col
                    >
                    <v-col align-self="center" md="4" class="centered">
                      <h4>Feels like</h4>
                      <span
                        >{{ weatherData.main.feels_like > 0 ? "+" : ""
                        }}{{ weatherData.main.feels_like }} &#8451;</span
                      >
                    </v-col>
                  </v-row>
                  <v-row justify="center">
                    <span
                      >{{ weatherData.main.temp_min > 0 ? "+" : ""
                      }}{{ weatherData.main.temp_min }} &#8451; ...
                      {{ weatherData.main.temp_max > 0 ? "+" : ""
                      }}{{ weatherData.main.temp_max }}
                      &#8451;
                    </span>
                  </v-row>
                </v-col>

                <v-divider vertical />

                <!-- Icon & description -->
                <v-col class="centered" align-self="center"
                  ><div>
                    <v-icon id="main-icon">{{
                      getIcon(weatherData.weather[0].icon)
                    }}</v-icon>
                  </div>
                  <div>
                    <span>{{ weatherData.weather[0].description }}</span>
                  </div></v-col
                >

                <v-divider vertical />

                <!-- Other details -->
                <v-col align-self="center"
                  ><v-row justify="center" class="detail-row"
                    ><v-icon>mdi-weather-windy</v-icon
                    ><span>{{ weatherData.wind.speed }} m/s</span
                    ><v-icon
                      v-bind:style="{
                        transform: `rotate(${weatherData.wind.deg + 180}deg)`
                      }"
                      >mdi-navigation</v-icon
                    ></v-row
                  >
                  <v-row justify="center" class="detail-row"
                    ><v-icon>mdi-water-percent</v-icon
                    ><span>{{ weatherData.main.humidity }} %</span></v-row
                  ><v-row justify="center" class="detail-row"
                    ><v-icon>mdi-gauge</v-icon
                    ><span>{{ weatherData.main.pressure }} mbar</span></v-row
                  ></v-col
                >
              </v-row>
              <v-row v-else justify="center">
                <LoadingSpinner />
              </v-row>
            </v-col>
          </v-row>
        </v-col>
      </v-row>

      <v-row>
        <v-divider />
      </v-row>

      <!-- Forecast -->
      <v-row justify="center">
        <v-col align="start">
          <v-row justify="center">
            <h2>Forecast</h2>
          </v-row>
          <v-row v-if="!loadingForecast">
            <v-col v-for="day in forecastData" :key="day.dt" align="center">
              <h4>{{ getDay(day.dt) }}</h4>
              <div>
                <v-icon class="forecast-icon">{{
                  getIcon(day.weather[0].icon)
                }}</v-icon>
              </div>
              <span
                >{{ day.main.temp > 0 ? "+" : ""
                }}{{ day.main.temp }} &#8451;</span
              >
            </v-col>
          </v-row>
          <v-row v-else justify="center"><LoadingSpinner /></v-row>
        </v-col>
      </v-row>
    </v-col>
  </v-container>
</template>

<script>
import axios from "axios";
import { format } from "date-fns";

import LoadingSpinner from "./LoadingSpinner";

const APPID = process.env.VUE_APP_WEATHER_APP_ID;

export default {
  name: "Weather",
  components: {
    LoadingSpinner
  },
  data: function() {
    return {
      weatherData: {},
      forecastData: [],
      date: format(new Date(), "dd.MM.yyyy HH:mm"),
      cities: [
        { name: "Espoo", id: 660158 },
        { name: "Hong Kong", id: 1819730 }
      ],
      selectedCity: 660158
    };
  },
  computed: {
    loadingWeather: function() {
      return Object.keys(this.weatherData).length < 1;
    },
    loadingForecast: function() {
      return this.forecastData.length < 1;
    }
  },
  created: async function() {
    await this.updateData();
  },
  methods: {
    updateData: async function() {
      this.weatherData = {};
      this.forecastData = [];
      const [weatherResponse, forecastResponse] = await Promise.all([
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?APPID=${APPID}&id=${this.selectedCity}&units=metric`
        ),
        axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?APPID=${APPID}&id=${this.selectedCity}&units=metric`
        )
      ]);
      if (weatherResponse.status < 400) {
        this.weatherData = weatherResponse.data;
      } else {
        console.log(weatherResponse);
      }
      if (forecastResponse.status < 400) {
        this.forecastData = forecastResponse.data.list.filter(
          (d, i) => i % 8 === 7
        );
      } else {
        console.log(forecastResponse);
      }
    },
    getDay: function(dt) {
      return format(new Date(dt * 1000), "EEE dd.MM.");
    },
    getIcon: function(text) {
      switch (text) {
        case "02n":
        case "04d":
          return "mdi-cloud";
        case "03d":
        case "03n":
        case "04n":
          return "mdi-weather-partly-cloudy";
        case "01d":
        case "01n":
          return "mdi-weather-sunny";
        case "10d":
        case "10n":
          return "mdi-weather-rainy";
        default:
          return "mdi-help-circle-outline";
      }
    }
  }
};
</script>

<style scoped>
#main-temp {
  font-size: 4em;
}

#main-icon {
  font-size: 8em;
}

.forecast-icon {
  font-size: 4em !important;
  margin: 0.5em 0;
}

.centered {
  text-align: center;
}

#citySelector {
  position: absolute;
  right: 2em;
  top: 2em;
}

.detail-row {
  margin: 1em 0;
  font-size: 2em;
}

.detail-row i {
  font-size: 1em;
  margin: 0 0.25em;
}
</style>
