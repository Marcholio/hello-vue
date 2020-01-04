<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" app clipped>
      <v-list dense>
        <v-list-item @click="weatherSelected">
          <v-list-item-action>
            <v-icon>mdi-weather-partly-cloudy</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Weather</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="otherSelected">
          <v-list-item-action>
            <v-icon>mdi-help-circle-outline</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Other stuff</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app clipped-left>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Hello Vue.js!</v-toolbar-title>
    </v-app-bar>

    <v-content>
      <Weather />
    </v-content>

    <v-footer app>
      <span>&copy; 2019</span>
    </v-footer>
  </v-app>
</template>

<script>
import Weather from "./components/Weather";

export default {
  name: "App",

  components: {
    Weather
  },

  data: () => ({
    drawer: null
  }),

  methods: {
    weatherSelected: function() {
      console.log("weather");
    },
    otherSelected: function() {
      console.log("other");
    }
  }
};
</script>
