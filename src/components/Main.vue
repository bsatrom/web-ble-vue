<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <v-list dense>
        <v-list-item @click>
          <v-list-item-action>
            <v-icon>mdi-home-circle</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Home</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click>
          <v-list-item-action>
            <v-icon>mdi-circle-double</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Beacons</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="indigo" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Particle JS API Demo</v-toolbar-title>
    </v-app-bar>

    <v-content>
      <v-snackbar v-model="success" color="success" multi-line top>
        <v-layout align-center wrap>{{successMsg}}</v-layout>
        <v-btn color="black" @click="success = false">Close</v-btn>
      </v-snackbar>
      <v-snackbar v-model="error" color="error" multi-line top>
        <v-layout align-center wrap>{{errorMsg}}</v-layout>
        <v-btn color="black" @click="error = false">Close</v-btn>
      </v-snackbar>

      <v-container fluid grid-list-md>
        <v-layout justify-center wrap>
          <v-flex text-center xs12>
            <v-data-table
              :headers="headers"
              :items="streamItems"
              :items-per-page="5"
              class="elevation-1"
              v-if="streamItems.length > 0"
            ></v-data-table>
          </v-flex>
          <v-flex text-center xs6 md2>
            <v-tooltip right>
              <template v-slot:activator="{ on }">
                <v-btn
                  color="indigo"
                  :href="source"
                  class="text-center"
                  dark
                  x-large
                  target="_blank"
                  @click="toggleLED"
                >
                  <v-icon large>mdi-alarm-light</v-icon>Toggle LED
                </v-btn>
              </template>

              <span>Click to toggle the device LED!</span>
            </v-tooltip>
          </v-flex>
          <v-flex text-center xs6 md2>
            <v-tooltip left>
              <template v-slot:activator="{ on }">
                <v-btn
                  color="indigo"
                  :href="source"
                  class="text-center"
                  dark
                  x-large
                  target="_blank"
                  @click="startDeviceStream"
                >
                  <v-icon large>mdi-playlist-play</v-icon>Start Stream
                </v-btn>
              </template>

              <span>Click to start the device stream!</span>
            </v-tooltip>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <v-footer color="indigo" app>
      <span class="white--text">&copy; Particle 2019</span>
    </v-footer>
  </v-app>
</template>

<script>
import Particle from "particle-api-js";
const particle = new Particle();
const device = process.env.VUE_APP_DEVICE_ID;
const token = process.env.VUE_APP_PARTICLE_TOKEN;

export default {
  props: {
    source: String
  },
  methods: {
    toggleLED: async function() {
      try {
        const toggle = await particle.callFunction({
          deviceId: device,
          name: "toggleLed",
          auth: token
        });

        this.successMsg = "LED Toggled";
        this.success = true;
      } catch (err) {
        this.errorMsg = "Function call error:" + err;
        this.error = true;
      }
    },
    startDeviceStream: function() {
      const that = this;

      particle
        .getEventStream({ deviceId: device, name: "env-vals", auth: token })
        .then(function(stream) {
          that.successMsg = "Stream Initialized";
          that.success = true;

          stream.on("event", function(data) {
            console.log("New Event: ", data.data);
            that.streamItems.push(JSON.parse(data.data));
          });

          stream.on("disconnect", function() {
            that.errorMsg = "Disconnected from Particle event stream";
            that.error = true;
          });

          stream.on("error", function(data) {
            that.errorMsg = "Stream Error: " + error;
            that.error = true;
          });
        });
    }
  },

  data: () => ({
    headers: [
      { text: "Temp", value: "temp" },
      { text: "Humidity", value: "humidity" },
      { text: "Light", value: "light" },
      { text: "Location Lat", value: "location.lat" },
      { text: "Location Long", value: "location.lon" }
    ],
    drawer: false,
    streamItems: [],
    error: false,
    success: false,
    errorMsg: null,
    successMsg: null
  })
};
</script>