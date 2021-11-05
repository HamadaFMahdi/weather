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
          @keyup.enter="getWeather"
          :error-messages="errorMessage"
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
            :src="imageURL"
            height="200px"
            :alt="`This is an image of ${city}`"
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
                {{cityDesc}}
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
    },
    errorMessage: '',
    imageURL: '',
    cityDesc: ''
  }),
  methods: {
    getWeather() {
      this.errorMessage = ''
      this.loading = 'pink'
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

          // fetch the image
          this.getImage()
          // fetch the desc
          this.getDesc()
        })
        .catch(err => {
          console.log(`This error occured: ${err}`)
          this.errorMessage = 'Please try again. If this error continues please contact us.'
          this.loading = null
        })
    },
    getImage() {
      fetch(`https://api.unsplash.com/search/photos/?client_id=#&query=${this.city}&content_filter=high`)
        .then(data => data.json())
        .then(data => {
          let numResults = data.results.length
          let randNum = Math.floor(Math.random() * numResults)
          this.imageURL = data.results[randNum].urls.small
        })
        .catch(err => {
          console.log(err)
          this.errorMessage = 'Unable to load image, please try again later.'
        })

    },
    getDesc() {
      fetch(`https://en.wikipedia.org/w/api.php?format=json&action=query&prop=extracts&exintro&explaintext&redirects=1&origin=*&titles=${this.city}`)
        .then(data => data.json())
        .then(data => {
          let pages = data.query.pages
          let id = Object.keys(pages)[0]
          this.cityDesc = pages[id].extract
        })
        .catch(err => {
          console.log(err)
          this.errorMessage = 'Unable to load city description.'
        })
    },
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

