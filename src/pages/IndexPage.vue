<template>
  <q-page class="flex column" :class="setBg">
    <div class="col q-pt-lg q-px-md flex" :class="$q.platform.is.desktop ? 'justify-center' : ''">
      <q-input v-model="search" placeholder="Search" class="search" dark borderless @keyup.enter="getWeatherBySearch">
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].description }}
        </div>
        <div class="text-h1 text-weight-th q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h3 relative-position degree">&deg;</span>
        </div>
      </div>
      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" alt="Fake img" />
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight thin">
          Quasar<br />
          Weather
        </div>
        <div class="col">
          <q-btn flat :ripple="{ center: true }" @click="getLocation">
            <q-icon left size="3em" name="my_location" />
            <div>Find my location</div>
          </q-btn>
        </div>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { QSpinnerFacebook, Platform } from 'quasar';

export default defineComponent({
  name: "IndexPage",

  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      long: null,
      apiKey: '826b150529c7562473f85d660947223f',
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
    };
  },
  computed: {
    setBg() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation() {
      navigator.geolocation.getCurrentPosition((position) => {
        console.log("position : ", position.coords.latitude);
        this.lat = position.coords.latitude;
        this.long = position.coords.longitude;
        this.getWeatherByCoords();
      });
    },
    getWeatherByCoords() {

        this.$q.loading.show({
          spinnerSize: 140,
          backgroundColor: 'purple',
          message: 'Chargement des données ...',
          messageColor: 'white'
        })
        this.$axios(
          `${this.apiUrl}?lat=${this.lat}&lon=${this.long}&appid=${this.apiKey}&units=metric`
        ).then((response) => {
          console.log(response);
          this.weatherData = response.data;
          this.$q.loading.hide()
        });
      
    },
    getWeatherBySearch() {
      if ((this.search !== '')) {
        this.$q.loading.show({
          spinnerSize: 140,
          backgroundColor: 'purple',
          message: 'Chargement des données ...',
          messageColor: 'white'
        })
        this.$axios(
          `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
        ).then((response) => {
          this.weatherData = response.data
          this.$q.loading.hide()
        })
        .catch((err) => {
          this.$q.loading.hide()
          this.$q.notify({
          'type' : 'warning',
          'message' : 'Veuillez rentrer une ville valide',
          'position':  'center',
          'closeBtn' : 'Fermer'
        });
        });
      } else {
        this.$q.notify({
          'type' : 'warning',
          'message' : 'Veuillez rentrer une ville',
          'position':  'center',
          'closeBtn' : 'Fermer'
        });
      }
    },

  },
});
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #fc5c7d, #6a82fb)
  &.bg-night
    background: linear-gradient(to top, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #fc5c7d, #6a82fb)
  .degree
    top: -40px

  .skyline
    flex: 0 0 100px
    background-image: url('../../public/skyline.png')
    background-size: contain
    background-position: center bottom
  
  .search
    @media (min-width: 1080px) 
      width: 50%
      text-align: center
    
</style>