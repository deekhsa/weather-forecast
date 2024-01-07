<template>
  <div>
    <section class="hero ">
      <div class="hero-body bucket">
        <div class="container">
          <div class="is-flex is-flex-direction-column">
            <p class="title is-5 has-text-weight-bold has-text-centered ">
                  Weather forecast  
            </p>
            <div class="columns is-desktop  is-flex is-align-items-center">
              <div class=" column">
                <div class="is-flex is-flex-direction-row is-align-items-center">
                  <figure class="image is-96x96 pr-2 is-hidden-mobile">
                  <img :src="require(`@/assets/buefy.png`)">
                </figure>
                <b-autocomplete
                    :expanded="true"
                    v-model="selectedLocation"
                    :data="locations"
                    :placeholder="'Enter your City'"
                    :open-on-focus="true"
                    @input="fetchLocations"
                    @select="fetchForcast"
                  ></b-autocomplete>

                </div>
              </div>
              <div class="column is-3 is-offset-5">
                <div class="is-flex is-flex-direction-row is-align-items-center">
                  <b-datepicker
                    placeholder=""
                    v-model="date"
                    icon="calendar-today"
                    locale="en-CA"
                    :min-date="minDate"
                    editable>
                  </b-datepicker>
                  <b-button type="is-dark" @click="filteredData">Filter</b-button>
                </div>
              </div>
            </div>
            <div class="is-flex flex-direction-row">
              <div class=" is-flex is-flex-direction-row is-justify-content-center" v-if="forecast && !filter">
              <div class="box m-3">
                <div class="has-text-centered mb-5">
                  <img :src="forecast.current.condition.icon" />
                  <p class="subtitle is-5 has-text-weight-bold ">
                    {{ forecast.current.condition.text }}
                </p>
                <p class="is-size-6 is-size-7-mobile has-text-weight-bold ">
                    <i class="fa-solid fa-droplet"></i>
                    {{ forecast.current.humidity }}%
                </p>
                <p class="is-size-6 is-size-7-mobile">
                  <i class="fas fa-wind"></i>
                    {{ forecast.current.wind_kph }}Kph, {{ forecast.current.wind_dir }}
                </p>
                <p class="is-size-6 is-size-7-mobile">
                    {{ forecast.current.temp_c }}&#176;C
                </p>
                </div>
                <div class="has-text-centered">
                  <div v-if="forecast">
                    <div class="columns is-multiline is-flex is-flex-direction-row is-justify-content-center">
                      <div v-for="day in forecast.forecast.forecastday" :key="day" class="p-5 ">
                        <img :src="day.day.condition.icon" />
                        <p class="subtitle is-6 has-text-weight-bold is-size-7-mobile">{{ day.day.condition.text }}</p>
                        <p class="is-size-6 is-size-7-mobile">{{ day.date }}</p>
                        <p class="is-size-6 is-size-7-mobile">{{ day.day.maxtemp_c }}&#176;C</p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              </div>
              <div class=" is-flex is-flex-direction-row is-justify-content-center" v-if="showFiltereddata">
                <div class="box m-3">
                  <div class="has-text-centered mb-5">
                      <img :src="filter.day.condition.icon" />
                      <p class="subtitle is-5 has-text-weight-bold ">
                        {{ filter.day.condition.text }}
                    </p>
                    <p class="is-size-6 is-size-7-mobile has-text-weight-bold ">
                        <i class="fa-solid fa-droplet"></i>
                        {{ filter.day.avghumidity }}%
                    </p>
                    <p class="is-size-6 is-size-7-mobile">
                      <i class="fas fa-wind"></i>
                        {{ filter.day.maxwind_kph }}Kph
                    </p>
                    <p class="is-size-6 is-size-7-mobile">
                        {{ filter.day.avgtemp_c }}&#176;C
                    </p>
                </div>
                  <div class="has-text-centered">
                    <div v-if="filter">
                      <div class="columns is-multiline is-flex is-flex-direction-row is-justify-content-center">
                        <div v-for="hour in filter.hour" :key="hour" class="p-5 ">
                          <img :src="hour.condition.icon" />
                          <p class="subtitle is-6 has-text-weight-bold is-size-7-mobile">{{ hour.condition.text }}</p>
                          <p class="is-size-6 is-size-7-mobile">{{ hour.time }}</p>
                          <p class="is-size-6 is-size-7-mobile">{{ hour.temp_c }}&#176;C</p>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="box m-3" v-if="forecast">
                <div class="has-text-centered mb-5">
                  <i class="fa-solid fa-smog fa-3x pb-4"></i>
                  <p class="subtitle is-5 is-size-6-mobile has-text-weight-bold ">
                    Air Quality
                </p>
                <p class="subtitle is-6 is-size-7-mobile"><b>Co: </b> {{ forecast.current.air_quality.co}}</p>
                <p class="subtitle is-6 is-size-7-mobile"><b>No2: </b> {{ forecast.current.air_quality.no2}}</p>
                <p class="subtitle is-6 is-size-7-mobile"><b>So2: </b> {{ forecast.current.air_quality.so2}}</p>
                <p class="subtitle is-6 is-size-7-mobile"><b>O3: </b> {{ forecast.current.air_quality.o3}}</p>
                <p class="subtitle is-6 is-size-7-mobile"><b>PM2.5: </b>{{ forecast.current.air_quality.pm2_5}}</p>
                <p class="subtitle is-6 is-size-7-mobile"><b>PM10: </b>{{ forecast.current.air_quality.pm10}}</p>

                </div>
              </div>

            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios';
import '@fortawesome/fontawesome-free/css/all.css'; 


export default {
  name: 'DefaultLayout',
  data () {
    return {
      selectedLocation: '',
      locations: [],
      forecast: null,
      date:null,
      filter:null,
      minDate: null,
      showFiltereddata:false 
    }
  },
  computed: {
  updatedMinDate() {
    return this.calculateMinDate();
  }
  },
  mounted() {
    this.minDate = this.updatedMinDate;
  },
  methods:{
    async fetchLocations(query) {
      this.showFiltereddata = false
      this.filter = null
      this.forecast = null
      try {
        const payload = {
          "lat": "28.6304",
          "lon": "77.2177",
          "query": query
        }
        const response = await axios.post(`https://webapi.magicpin.in/ultron-web/searchLocality/`,payload);
        this.locations = response.data.apiResponse.localities.map(location => ({
          value: `${location.address}`,
        }));
      } catch (error) {
        console.error('Error fetching locations:', error);
      }
    },
    calculateMinDate() {
        return new Date(); 
      },
    async fetchForcast() {
      try {
        const response = await axios.post(`http://api.weatherapi.com/v1/forecast.json?key=c1c0df0517e64de499163959240601&q=${this.selectedLocation}&days=7&aqi=yes&alerts=no`);
        console.log("data", response.data);
        this.forecast = response.data;
      } catch (error) {
        console.error("Error fetching forecast:", error);
      }
    },
    async filteredData(){
      try {
        if(!this.selectedLocation){
          this.$buefy.toast.open('Please Enter City')
        }else if(!this.date){
          this.$buefy.toast.open('Please Enter date')
        }else{
          const formatDate = this.date.toISOString().split('T')[0]; 
          const response = await axios.get(`http://api.weatherapi.com/v1/future.json?key=c1c0df0517e64de499163959240601&q=${this.selectedLocation}&dt=${formatDate}`);
            this.filter = response.data.forecast.forecastday[0];
            console.log("data",this.forecast,this.filter);
            this.showFiltereddata = true
        }
      } catch (error) {
            this.$buefy.toast.open('Please Enter any date that is 14 days ahead and within 300 days');
      }
    }
  }
}
</script>


