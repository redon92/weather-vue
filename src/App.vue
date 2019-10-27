<template>
  <div id="app">
    <h1>FieldClimate Test</h1>

    <div class="row text-center">
      <select class="form-control" v-model="resolution">
        <option hidden value="">Resolution</option>
        <option v-for="resolutions in givenResolutions" :value="resolutions">{{resolutions}}</option>
      </select>
      <select class="form-control" v-model="period">
        <option hidden value="">Period</option>
        <option v-for="periods in givenPeriods" :value="periods.value">{{periods.title}}</option>
      </select>
      <button class="btn" @click="getData(resolution, period)">Submit</button>
    </div>

    <div id="chart">
      <weatherChart :chart-data="chartData" :options="chartOptions"></weatherChart>
    </div>

    <div>
      <weatherTable :tableData="fetchedData"></weatherTable>
    </div>
  </div>
</template>

<script>
  const CryptoJS = require('crypto-js');
  import apiValues from './apiValues.js';
  import weatherChart from './components/weatherChart';
  import weatherTable from './components/weatherTable';

export default {
  name: 'app',
  components: {
    weatherChart,
    weatherTable
  },
  mounted() {
    let self=this;
    // constant values from apiValues.js
    self.apiValues = apiValues;
    // initialize graph when page loads with raw resolution for 24h
    self.getData('raw', '24h');
  },
  data() {
    return {
      apiValues: {},
      fetchedData:{},
      resolution:'',
      period:'',
      givenResolutions:['raw', 'hourly', 'daily'],
      givenPeriods:[
        {title:'24 hours',value: '24h'},
        {title: '2 days', value:'2d'},
        {title: '7 days', value:'7d'},
        {title: '30 days', value:'30d'}
      ],

      chartData:{
        labels:{},
        datasets: []
      },
      chartOptions:{
        responsive: true,
        // removes aspectRatio so it takes the height we define
        maintainAspectRatio: false,

      },
    };
  },
  methods: {

    getData(resolution, period) {
      if (resolution !== '' && period !== '') {
        let self = this;
        let request = "/data/00000264/" + resolution + "/last/" + period;
        let Url = self.apiValues.baseurl + request;

        let timestamp = new Date().toUTCString();  //Wed, 09 Aug 2017 20:32:38 GMT
        let content_to_sign = self.apiValues.method + request + timestamp + self.apiValues.public_key;
        let signature = CryptoJS.HmacSHA256(content_to_sign, self.apiValues.private_key);
        let hmac_str = 'hmac ' + self.apiValues.public_key + ':' + signature;
        let parameters = {
          headers: {
            "Accept": "application/json",
            "Authorization": hmac_str,
            "Request-Date": timestamp
          },
          method: self.apiValues.method
        };
        // gets data from api, and populates table and graph
        self.getFetch(Url, parameters);
      }
    },

    getFetch(url, parameters) {
      let self=this;
      let gottenData;
      fetch(url, parameters)
        .then(data => {
          return data.json()
        }).then(function (response) {
          // assigns the fetched data to fetchedData
          self.fetchedData=response;
          // populate the chartData object the submit button is pressed, and then it's passed reactively to weatherChart,
          // which generates the chart
          self.chartData = {
            labels: self.fetchedData.dates,
            datasets: [
              {
                label: "Solar Panel",
                borderColor: "#f8e600",
                fill:false,
                data: self.fetchedData.data[0].values.last
              },
              {
                label: "Battery",
                borderColor: "#67cd44",
                fill:false,
                data: self.fetchedData.data[2].values.last
              },
              {
                label: "Temp Min",
                borderColor: "#0b49f8",
                fill:false,
                data: self.fetchedData.data[5].values.min
              },
              {
                label: "Temp Avg",
                borderColor: "#ccc7c9",
                fill:false,
                data: self.fetchedData.data[5].values.avg
              },
              {
                label: "Temp Max",
                borderColor: "#f87979",
                fill:false,
                data: self.fetchedData.data[5].values.max
              },
            ]
          };
      }).catch(error => console.log(error));
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 25px;
}
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
th, td {
  padding: 5px;
  text-align: left;
}
#chart{
    max-height: 500px;
  }
.form-group{

}
.form-control {
  /*display: block;*/
  /*width: 100%;*/
  margin-right: 10px;
  margin-left: 10px;
  padding: .375rem .75rem;
  font-size: 1rem;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #495057;
  border-radius: .25rem;
  transition: border-color .15s ease-in-out,box-shadow .15s ease-in-out;
}
.btn {
  margin-right: 10px;
  margin-left: 10px;
  display: inline-block;
  /*font-weight: 400;*/
  text-align: center;
  white-space: nowrap;
  /*vertical-align: middle;*/
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  border: 1px solid #495057;
  padding: .19rem .75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: .25rem;
  cursor: pointer;
  transition: color .15s ease-in-out,background-color .15s ease-in-out,border-color .15s ease-in-out,box-shadow .15s ease-in-out;
}
  .row{
    margin-top: 25px;
    margin-bottom: 25px;
  }
</style>
