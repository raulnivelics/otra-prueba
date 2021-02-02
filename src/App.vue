<template>
 <body>
      <div id="app"> 
        <h1 class="translate">{{name_city}}</h1>
        <h3>{{overcast}}</h3>
        <div id="weather">
        
          <img src="./assets/images/icono_sol.png" style="width: 200px;">
          <span class="temperature">{{currentTemp}}°</span><br>
          <span id="temp-values">Min {{minTemp}}° <br> Max {{maxTemp}}°</span>
        </div>
        <div id="info">
          <img class="icon" src="./assets/images/amanecer.png" style="width:25px"> {{sunrise}}
          <img class="icon" src="./assets/images/atardecer.png" style="width:25px"> {{sunset}}
          <img class="icon" src="./assets/images/humedad2.png" style="width:25px"> {{humidity}}
          <img class="icon" src="./assets/images/presion.png" style="width:25px"> {{pressure}}
          <img class="icon" src="./assets/images/viento.png" style="width:25px"> {{wind}}
        </div>
        <div >
          <input type="button" class="button" @click="OpenModal" value="Actualizar Ciudad"/>
        </div> 
      </div>
        <div v-if="modalCity">
          <div id="myModal" class="modal">
           <div class="modal-content">
            <div class="modal-header">
              <span class="close" @click="closeModal">&times;</span>
              <h2>Ingrese el nombre de la ciudad para actualizar la información del clima</h2>
            </div>
            <div class="modal-body">
              <div class="form-group" style='text-align:center'>
              <label>Ciudad</label>
              <input type="text" class="form-control" v-model="input_city" />
              </div>
              <br />
              <div align="center">
              <input type="button" id="procesar_ciudad" class="button" @click="submitData"  value="Procesar"/>
              </div>
            </div>
          </div>
          </div>
          </div>
  
     <div id="grafics">   
     <hr>   
     <h1 style="text-align:center">Gráfica de temperatura de los próximos 5 días</h1>
     <div id="day1">
     <GChart
      type="ColumnChart"
      :data="chartData"
      :options="chartOptions"
    />
     </div>
    </div>
    </body>
</template>

<script>
import Vue from 'vue';
import axios from "axios";
import moment from "moment";
import {GChart} from "vue-google-charts";
import VueGtm from 'vue-gtm';

Vue.use(VueGtm, {
  enabled: true,
  debug: true,
 });
 
export default {
  name: 'app',
  components: {
    GChart
  },
  data () {
    return {
      currentTemp: '',
      minTemp: '',
      maxTemp:'',
      sunrise: '',
      sunset: '',
      pressure: '',
      humidity: '',
      wind: '',
      overcast: '', 
      icon: '',
      city: 'medellin',
      name_city: '',
      modalCity : false,
      lat:'6.2518',
      lon:'-75.5636',
      chartData: [[]
        ],
        chartOptions: {
          chart: {
            title: 'Gráfica de Temperatura de los próximos 5 días',
          }
        }
    }
  },
   methods: {
      getWeather() {
        let url = "http://api.openweathermap.org/data/2.5/weather?q="+this.city+"&lang=es&units=metric&APPID=e2f24e73b812a4242d9bdfba8c653ce0";
        
        axios
          .get(url)
          .then(response => {
            this.currentTemp = response.data.main.temp;
            this.minTemp = response.data.main.temp_min;
            this.maxTemp = response.data.main.temp_max;
            this.pressure = response.data.main.pressure;
            this.humidity = response.data.main.humidity + '%';
            this.wind = response.data.wind.speed + 'm/s';
            this.name_city = response.data.name;
            this.overcast = response.data.weather[0].description;
            this.icon = "images/" + response.data.weather[0].icon.slice(0, 2) + ".svg";
            this.sunrise = moment.unix(response.data.sys.sunrise).format('hh:mm A');
            this.sunset = moment.unix(response.data.sys.sunset).format('hh:mm A');
            this.lon = response.data.coord.lon;
            this.lat = response.data.coord.lat;

            this.$gtm.trackEvent({
              event: 'ga_event',
              category: "Aplicativo",
              action: this.name_city,
              label: this.currentTemp
           });
        })
        .catch(error => {
          console.log(error);
        });
      },
      OpenModal() {
        this.modalCity = true;
        this.input_city = '';
      },
      submitData(){
        this.city = this.input_city;
        this.modalCity = false;
        this.getWeather();
      },
      closeModal(){
        this.modalCity = false;
      },
      getDataGrafics(){
      let url = "https://api.openweathermap.org/data/2.5/onecall?lat="+this.lat+"&lon="+this.lon+"&lang=es&appid=e2f24e73b812a4242d9bdfba8c653ce0";
          axios
            .get(url)
            .then(response => {
              var day1 = moment.unix(response.data.daily[1].dt).locale('es').format('dddd')+" - "+response.data.daily[1].weather[0].description;
              var day2 = moment.unix(response.data.daily[2].dt).locale('es').format('dddd')+" - "+response.data.daily[2].weather[0].description;
              var day3 = moment.unix(response.data.daily[3].dt).locale('es').format('dddd')+" - "+response.data.daily[3].weather[0].description;
              var day4 = moment.unix(response.data.daily[4].dt).locale('es').format('dddd')+" - "+response.data.daily[4].weather[0].description;
              var day5 = moment.unix(response.data.daily[5].dt).locale('es').format('dddd')+" - "+response.data.daily[5].weather[0].description;
              this.chartData = [['Día', '° Min', '° Max'],
              [day1, response.data.daily[1].temp.min, response.data.daily[1].temp.max],
              [day2, response.data.daily[2].temp.min, response.data.daily[2].temp.max],
              [day3, response.data.daily[3].temp.min, response.data.daily[3].temp.max],
              [day4, response.data.daily[4].temp.min, response.data.daily[4].temp.max],
              [day5, response.data.daily[5].temp.min, response.data.daily[5].temp.max]];
              })
          .catch(error => {
            console.log(error);
          });
      
    },
  
   },
   
   beforeMount() {
      Vue.use(VueGtm, {
        enabled: true,
        debug: true,
      });
      this.getWeather();
      this.getDataGrafics();
    },
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
  margin-top: 60px;
  width:    470px;
  height:   400px;
  /*position: absolute;*/
  left:     35%;
  background: url(./assets/images/fondo.png) no-repeat;
  margin: 0 auto;
}

h1, h2 {
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
#weather {
  padding: 15px;
  vertical-align: middle;
  margin-left:-190px;
}
#temp-values {
  text-align: right;
  text-justify: distribute;
  display: block;
  position: relative;
  top: -220px;
}

#info {
  padding-left: 20px;
  position: relative;
  top: -20px;
}
.temperature {
  font-family: 'Vast Shadow', cursive;
  font-size: 40px;
  vertical-align: top;
  position: absolute;
  margin-top: 75px;
}
.button {
  margin-top: 20px;
  background-color: #383d75;
  border: none;
  color: white;
  padding: 10px 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  border-radius: 5px;
}
.modal {
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  padding-top: 100px; /* Location of the box */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
}

.modal-content {
  position: relative;
  background-color: #fefefe;
  color:black;
  margin: auto;
  padding: 0;
  border: 1px solid #888;
  width: 40%;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2),0 6px 20px 0 rgba(0,0,0,0.19);
  -webkit-animation-name: animatetop;
  -webkit-animation-duration: 0.4s;
  animation-name: animatetop;
  animation-duration: 0.4s
}

.close {
  color: white;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}

.modal-header {
  padding: 2px 16px;
  background-color: #383d75;
  color: white;
}

.modal-body {padding: 2px 16px;}

#grafics{
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
}
</style>
