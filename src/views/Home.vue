<template>
  <v-container>

    <v-btn @click="createChart('canvas-chart', patriarca)">Cargar datos</v-btn>

    <v-card class="mt-7 pa-3">
      <canvas id="canvas-chart"></canvas>
    </v-card>

    <h5 v-for="(item, index) of today" :key="index">{{item.date}} - {{item.feel}} - {{item.plus}}</h5>

  </v-container>
</template>

<script>
  import Chart from 'chart.js';

export default {
  data: () => ({
    i: 0,
    today: [],
    patriarca: {
        type: 'line',
        data: {
          labels: [],
          datasets: [
            {
              label: 'Hoy',
              data: [],
              backgroundColor: [
                'rgba(54,73,93,.3)'
              ],
              borderColor: [
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
                '#36495d',
              ],
              borderWidth: 3
            }
          ]
        },
        options: {
          responsive: true,
          responsiveAnimationDuration: 0,
          lineTension: 1,
          aspectRatio: 4,
          scales: {
            yAxes: [{
              ticks: {
                beginAtZero: true,
                padding: 25,
              }
            }]
          }
        }
    }
  }),
  methods: {
    async createChart(chartId, chartData) {
      await this.dope()
      const ctx = await document.getElementById(chartId);
      const myChart = await new Chart(ctx, {
        type: chartData.type,
        data: chartData.data,
        options: chartData.options,
        myChart: myChart
      });
    },
    async dope() {
      await this.axios.get('/feel')
        .then(res => {
          this.today = res.data
          this.today.forEach(element => {
            this.patriarca.data.labels.push(element.date.substring(0,10))
            this.patriarca.data.datasets[this.i].data.push(element.feel)
          })
          this.i++
        })
        .catch(e => {
          console.log(e.response)
        })
    }
  }
}
</script>