<template>
  <q-layout view="lHh Lpr lFf">

    <q-page class="flex flex-center" :class="bgClass">
      
      <div class="col q-pt-lg q-px-md text-white">
          <q-input
                v-model="search"
                label="Search"
                @keyup.enter="getWeatherBySearch"
                dark 
                borderless
          >

          <template v-slot:before class="text-white">
            <q-icon name="place" @click="getLocation" class="cursor-pointer"/>
          </template>

          <template v-slot:append class="text-white">
            <q-btn 
              icon="search" 
              @click="getWeatherBySearch" 
              class="cursor-pointer"
              dense
              flat
              round 
            />
          </template>

         </q-input>
        

        <template v-if="weatherData != null">
          
          <div class="col text-white text-center">
            <div class="text-h4 text-weight-light">
              {{ weatherData.name }}
            </div>
            <div class="text-h6 text-weight-light">
              {{ weatherData.weather[0].main }}
            </div>
            <div class="text-h1 text-weight-thin q-my-lg">
              <span>
                {{
                  Math.round(weatherData.main.temp)
                }}
              </span>
              <span>
                &deg;C
              </span>
            </div>
          </div>

          <div class="col text-center">
            <img :src="`http://openweathermap.org/img/wn/${ weatherData.weather[0].icon }@2x.png`" alt="">
          </div>
        </template>

        <template v-if="weatherData == null">
          <div class="col column text-center text-gray text-weight-thin">
            <div class="col text-h2 text-weight-thin">
              Weather
              <br>
              App
            </div>
          </div>
        </template>

        <q-btn
          @click="getLocation"
          class="col q-btnwet"
          flat
          size="xl"
         >
          <q-icon left name="cloud" />
          <label style="font-size: 17px">
            Check Weather
          </label>
         </q-btn>

      </div>
    </q-page>
  </q-layout>
</template>

<script>
import axios from 'axios'

export default {
  name: 'PageIndex',
  data () {
    return {
      search: ' ',
      weatherData: null,
      lat: null,
      lon: null,
      appUrl: 'https://api.openweathermap.org/data/2.5/weather'
    }
  },
  computed: {
    bgClass () {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation () {
      this.$q.loading.show()

      if (this.$q.platform.is.electron) {
        axios('https://freegeoip.app/json/')
        .then( response => {
              this.lat = response.data.latitude
              this.lon = response.data.longitude
              this.getWeatherByCoords() 
            }
        )    

      } else {
        navigator.geolocation.getCurrentPosition (
          position => {
            this.lat = position.coords.latitude
            this.lon = position.coords.longitude
            this.getWeatherByCoords() 
          }
        )
      }
      
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      axios(
        `${ this.appUrl }?lat=${this.lat}&lon=${this.lon}&appid=8fb5441c0731f2a01434908aaf653fbb&units=metric`
      ).then( response => {
              this.weatherData = response.data
              this.$q.loading.hide()
            }
      )
    },
    getWeatherBySearch() {
      this.$q.loading.show()
      axios(
        `${ this.appUrl }?q=${ this.search }&appid=8fb5441c0731f2a01434908aaf653fbb&units=metric`
      ).then( response => {
              this.weatherData = response.data
              this.$q.loading.hide()
            }
      )
      this.search = ''
    }
  }
}
</script>

<style lang="scss">
  .q-page {
     background: linear-gradient(to bottom, #136a8a, #267871)
  }
  .q-page.bg-night {
    background: linear-gradient(to bottom, #232526, #414345)
  }
  .q-page.bg-day {
    background: linear-gradient(to bottom, #00b4db, #0083b0)
  }
  .q-btnwet {
    display: flex;
    width: 300px;
    margin: 20px auto;
  }
</style>
