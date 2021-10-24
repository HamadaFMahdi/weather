<template>
  <v-app>
    <v-row>
      <v-col
        cols="12"
        class="px-12 mt-12"
      >
        <v-text-field
          v-model="city"
          label="Search for your city..."
          solo
          clearable
          prepend-inner-icon="mdi-domain"
          append-icon="mdi-map-marker"
          background-color="indigo lighten-1"
          dark
          :loading="loading"
          @keyup.enter="getData"
        ></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col 
        cols="12"
      >
        <v-card
          v-show="showCard"
          class="mx-auto"
          max-width="344"
          color="indigo lighten-1"
          dark
        >
          <v-img
            src="https://cdn.vuetifyjs.com/images/cards/sunshine.jpg"
            height="200px"
          ></v-img>

          <v-card-title>
            {{city}}
          </v-card-title>

          <v-card-subtitle>
            {{weather.temp}}°C - {{weather.main}}, {{weather.desc}}
          </v-card-subtitle>

          <v-card-actions>
            <v-spacer>
            </v-spacer>
            <v-btn
              icon
              @click="showCityDesc = !showCityDesc"
            >
              <v-icon>{{ showCityDesc ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
            </v-btn>
          </v-card-actions>

          <v-expand-transition>
            <div v-show="showCityDesc">
              <v-divider></v-divider>

              <v-card-text>
                London is a great place (generally speaking).
              </v-card-text>
            </div>
          </v-expand-transition>
        </v-card>

      </v-col>
    </v-row>
  </v-app>
</template>

<script>



export default {
  name: 'App',

  components: {
  },

  data: () => ({
    city: "",
    showCityDesc: false,
    showCard: false,
    loading: null,
    weather: {
      temp: '',
      main: '',
      desc: '',
    }
  }),
  methods: {
    getData() {
      this.loading = 'white'
      fetch(`https://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=4d30afa58f6f935d861edecad3639cda`)
        .then(data => data.json())
        .then(data => {
          this.weather.temp = (data.main.temp - 273.15).toFixed(2)
          this.weather.main = data.weather[0].main
          this.weather.desc = data.weather[0].description
          setTimeout(() => {
            this.loading = null
            this.showCard = true
          },500)

        })
    }
  }
};
</script>

<style>
  .red-b {
    border: 2px red solid;
  }
  .green-b {
    border: 2px green solid;
  }
</style>

