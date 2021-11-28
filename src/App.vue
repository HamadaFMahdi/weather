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
          @click:append="getGeoLocation"
          background-color="indigo lighten-1"
          dark
          :loading="loading"
          @keyup.enter="getWeatherMain"
          :error-messages="errorMessage"
        ></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col 
        cols="12"
      >

        <!-- Init State -->
        <v-skeleton-loader
          v-show="!showCard"
          class="mx-auto"
          elevation="6"
          width="344"
          height="350"
          type="card"
        >
        </v-skeleton-loader>

        <!-- Filled state -->
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
            {{cityFormatted}}
          </v-card-title>

          <v-card-subtitle>
            {{weather.temp}}°C - {{weather.main}}, {{weather.desc}}
          </v-card-subtitle>

          <v-card-actions>
            <v-spacer>
            </v-spacer>
            <v-btn icon @click="saveCity">
              <v-icon>mdi-heart</v-icon>
            </v-btn>
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
                {{ cityDesc }}
                <br>
                <br>
                <v-btn small rounded color="white">
                   <a :href="cityWikiLink" target="_blank"> Read more </a>
                </v-btn>
              </v-card-text>
            </div>
          </v-expand-transition>
        </v-card>

      </v-col>
    </v-row>
    <v-row class="px-2">
      <v-col v-if="favouritedCities.length" cols="12" style="text-align: center;">
        <v-divider class="my-3"></v-divider>
        Your favourited cities:
      </v-col>
      <v-col v-else cols="12" style="text-align: center;">
        <v-divider class="my-3"></v-divider>
        Your favourited cities will appear here
      </v-col>
      <v-col
        v-for="favCity in favouritedCities"
        :key="favCity.cityFormatted"        
        cols="3"
      >
        <v-card
          class="mx-auto"
          max-width="344"
          color="indigo lighten-1"
          dark
        >
          <v-img
            :src="favCity.imgURL"
            height="200px"
            :alt="`This is an image of ${favCity.cityFormatted}`"
          ></v-img>

          <v-card-title>
            {{favCity.cityFormatted}}
          </v-card-title>

          <v-card-subtitle>
            {{favCity.weather.temp}}°C - {{favCity.weather.main}}, {{favCity.weather.desc}}
          </v-card-subtitle>

          <v-card-actions>
            <v-spacer>
            </v-spacer>
            <v-btn icon @click="saveCity">
              <v-icon>mdi-heart</v-icon>
            </v-btn>
            <v-btn
              icon
              @click="favCity.cityDescOpen = !favCity.cityDescOpen"
            >
              <v-icon>{{ favCity.cityDescOpen ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
            </v-btn>
          </v-card-actions>

          <v-expand-transition>
            <div v-show="favCity.cityDescOpen">
              <v-divider></v-divider>

              <v-card-text>
                {{ favCity.cityDesc }}
                <br>
                <br>
                <v-btn small rounded color="white">
                   <a :href="favCity.cityWikiLink" target="_blank"> Read more </a>
                </v-btn>
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
    cityDesc: '',
    cityWikiLink: '',
    favouritedCities: []
  }),
  mounted: function() {
    console.log("I was mounted")
    this.getfavouritedCities()
  },
  computed: {
    cityFormatted() {
      let cityTrimmed = this.city.trim()
      if(cityTrimmed) {
        return cityTrimmed[0].toUpperCase() + cityTrimmed.slice(1).toLowerCase()
      }
      return ""
    },
  },
  methods: {
    getfavouritedCities() {
      let cities = localStorage.getItem('cities')
      if(cities) {
        // 
        cities = cities.split(',')
        let allData = []
        for(let city of cities) {
          let cityData = {
            cityFormatted: city,
            cityDescOpen: false
          }
          // run all API calls
          this.getWeather(false, false, city)
            .then(data => {
              cityData.weather = {
                temp: (data.main.temp - 273.15).toFixed(2),
                main: data.weather[0].main,
                desc: data.weather[0].description,
              }

              this.getImage(city)
                .then(data => {
                  cityData.imgURL = data

                  this.getDesc(city)
                    .then(data => {
                      cityData.cityWikiLink = data[0]
                      cityData.cityDesc = data[1]
                      allData.push(cityData)
                    })
                })

            })
        }
        // this.favouritedCities = [
        //   {cityFormatted: 'London', weather: {temp: '1', main: 'a', desc: 'd',}, cityDesc: 'a nice city', cityWikiLink: 'google.com', imgURL: 'https://i.insider.com/5b9137e10ce5f5b27e8b4a0c?width=1000&format=jpeg&auto=webp'},
        //   {cityFormatted: 'Berlin', weather: {temp: '2', main: 'b', desc: 'e',}, cityDesc: 'a nice city', cityWikiLink: 'google.com', imgURL: 'https://i.insider.com/5b9137e10ce5f5b27e8b4a0c?width=1000&format=jpeg&auto=webp'},
        //   {cityFormatted: 'Tokyo', weather: {temp: '3', main: 'c', desc: 'f',}, cityDesc: 'a nice city', cityWikiLink: 'google.com', imgURL: 'https://i.insider.com/5b9137e10ce5f5b27e8b4a0c?width=1000&format=jpeg&auto=webp'},
        //   {cityFormatted: 'Aden', weather: {temp: '3', main: 'c', desc: 'f',}, cityDesc: 'a nice city', cityWikiLink: 'google.com', imgURL: 'https://i.insider.com/5b9137e10ce5f5b27e8b4a0c?width=1000&format=jpeg&auto=webp'},
        // ]
        this.favouritedCities = allData
      } else {
        this.favouritedCities = []
      }
    },
    getWeather(lat, long, cityFormatted) {
      let link;
      if(long) {
        link = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${long}&appid=4d30afa58f6f935d861edecad3639cda`
      } else {
        link = `https://api.openweathermap.org/data/2.5/weather?q=${cityFormatted}&appid=4d30afa58f6f935d861edecad3639cda`
      }
      return fetch(link)
        .then(data => data.json())
    },
    getWeatherMain(lat, long) {
      this.errorMessage = ''
      this.loading = 'pink'
      this.getWeather(lat, long, this.cityFormatted)
        .then(data => {
          this.weather = {
                temp: (data.main.temp - 273.15).toFixed(2),
                main: data.weather[0].main,
                desc: data.weather[0].description,
          }
          if(long) {
            this.city = data.name
          }
          setTimeout(() => {
            this.loading = null
            this.showCard = true
          },500)
    
          // fetch the image
          this.getMainImage()
          // fetch the desc
          this.getMainDesc()
        })
        .catch(err => {
          console.log(`This error occured: ${err}`)
          this.errorMessage = 'Please try again. If this error continues please contact us.'
          this.loading = null
        })

    },
    getImage(city) {
      return fetch(`https://api.unsplash.com/search/photos/?client_id=UNSPASH_API&query=${city}&content_filter=high`)
        .then(data => data.json())
        .then(data => {
          let numResults = data.results.length
          let randNum = Math.floor(Math.random() * numResults)
          return data.results[randNum].urls.small
        })
    },
    getMainImage() {
      this.getImage(this.cityFormatted)
        .then(data => {
          this.imageURL = data
        })
        .catch(err => {
          console.log(err)
          this.errorMessage = 'Unable to load image, please try again later.'
        })
    },
    getDesc(city) {
      return fetch(`https://en.wikipedia.org/w/api.php?format=json&action=query&prop=extracts&exintro&explaintext&redirects=1&origin=*&titles=${city}`)
        .then(data => data.json())
        .then(data => {
          let pages = data.query.pages
          let id = Object.keys(pages)[0]
          let desc = pages[id].extract
          if(desc.length > 250) {
            desc = this.shortenDesc(desc)
          }
          return ['https://en.wikipedia.org/?curid=' + id, desc]
        })
    },
    getMainDesc() {
      this.getDesc(this.cityFormatted)
        .then((data)=>{
          this.cityWikiLink = data[0]
          this.cityDesc = data[1]
        })
        .catch(err => {
          console.log(err)
          this.errorMessage = 'Unable to load city description.'
        })
    },
    shortenDesc(desc) {
      return desc.slice(0, 250) + '...'
    },
    getGeoLocation() {
      let options = {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      };

      let success = (pos) => {
        let crd = pos.coords;
        this.getWeatherMain(crd.latitude, crd.longitude)
      }

      let error = (err) => {
        console.log(`ERROR(${err.code}): ${err.message}`);
        this.errorMessage = 'Unable to get location.'
        this.loading = null
      }
      this.loading = 'pink'
      navigator.geolocation.getCurrentPosition(success, error, options);
    },
    saveCity() {
      let cities = localStorage.getItem('cities');
      if(cities) {
        let citiesList = cities.split(',')
        if(citiesList.length === 4) {
          console.log('already 4')
          return 
        }
        let cityFormatted = this.cityFormatted
        if(citiesList.includes(cityFormatted)) {
          console.log('already in there')
          return 
        }
        citiesList.push(cityFormatted)
        localStorage.setItem('cities', citiesList);
      } else {
        localStorage.setItem('cities', [this.cityFormatted])
      }
      this.getfavouritedCities()
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

