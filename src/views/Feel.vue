<template>
<v-container>

  <v-layout>
    <v-btn color="primary" class="mr-4" @click="generarGrafico(4)">Año</v-btn>
    <v-btn color="primary" class="mr-4" @click="generarGrafico(3)">Mes</v-btn>
    <v-btn color="primary" class="mr-4" @click="generarGrafico(2)">Día</v-btn>
    <v-btn color="primary" class="mr-4" @click="generarGrafico(1)">últimos</v-btn>
  </v-layout>

  <v-layout class="mt-4" v-show="navGrafico!=0">
    <v-flex xs12>
      <v-card class="pa-xs-2 pa-sm-6">
        <canvas id="canvas-chart"></canvas>
      </v-card>
    </v-flex>
  </v-layout>

  <v-data-table
    :headers="headers"
    :items="feels"
    :items-per-page="10"
    sort-by="date"
    :sort-desc="true"
    class="elevation-1 mt-5"
  > 
    <template v-slot:top>
      <v-toolbar flat color="white">
        
        <v-toolbar-title>Hay tabla</v-toolbar-title>

        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>

        <v-spacer></v-spacer>

        <v-dialog v-model="dialog" max-width="500px">

          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">Añadir</v-btn>
          </template>

          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="3">
                    <v-text-field type="number" min="1" max="10" v-model="editedItem.feel" label="1-10" autofocus></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="9">
                    <v-text-field v-model="editedItem.plus" label="Descripción" @keyup.enter="save"></v-text-field>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" xs="6" sm="4" v-for="(item, index) of tareas" :key="index"><v-switch :label="item"></v-switch></v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
              <v-btn color="blue darken-1" text @click="save">Guardar</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="lsFeels">Traer Data</v-btn>
    </template>
  </v-data-table>

  </v-container>
</template>

<script>
  import Chart from 'chart.js';
  import { mapState } from 'vuex'

  export default {
    data: () => ({
      patriarca: {
          type: 'line',
          data: {
            labels: [],
            datasets: [
              {
                label: 'Actual',
                data: [],
                backgroundColor: ['rgba(0,0,255,.3)'],
                borderColor: [],
                borderWidth: 3
              },
              {
                label: 'Actual-1',
                data: [],
                backgroundColor: ['rgba(255,255,0,.3)'],
                borderColor: [],
                borderWidth: 3
              },
              {
                label: 'Actual-2',
                data: [],
                backgroundColor: ['rgba(255,0,0,.3)'],
                borderColor: [],
                borderWidth: 3
              }
            ]
          },
          options: {
            responsive: true,
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
      },
      navGrafico: 0,
      feels: [],
      dialog: false,
      headers: [
        {
          text: 'Fecha',
          align: 'start',
          sortable: true,
          value: 'date',
        },
        { text: 'Feel', value: 'feel' },
        { text: 'Descripción', value: 'plus' },
        { text: 'Verbo', value: 'verb' },
        { text: 'Acciones', value: 'actions', sortable: false },
      ],
      editedIndex: -1,
      editedItem: {},
      defaultItem: {
        feel: Number,
        plus: ''
      }
    }),
    computed: {
      ...mapState(['tareas']),
      formTitle () {
        return this.editedIndex === -1 ? 'Añadir Feel' : 'Editar Feel'
      }
    },
    watch: {
      dialog (val) {
        val || this.close()
      }
    },
    created () {
      this.lsFeels()
    },
    methods: {
      async createChart(chartId, chartData) {
        const ctx = await document.getElementById(chartId);
        const myChart = await new Chart(ctx, {
          type: chartData.type,
          data: chartData.data,
          options: chartData.options,
          myChart: myChart
        });
      },
      async lsFeels() {
        await this.axios.get('/feel')
          .then(res => {
            this.feels = res.data
          })
          .catch(e => {
            console.log(e.response)
          })
      },
      editItem (item) {
        this.editedIndex = this.feels.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },
      deleteItem (item) {
        const indexArray = this.feels.indexOf(item)
        const indexDb = item._id
        confirm('Eliminar definitivamente?') && this.axios.delete(`/feel/${indexDb}`)
        .then(this.feels.splice(indexArray, 1))
        .catch(e => {
          console.log(e.response)
        })
      },
      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },
      save () {
        if (this.editedIndex > -1) {
          const index = this.editedIndex
          this.axios.put(`/feel/${this.editedItem._id}`, this.editedItem)
            .then(res => {
              Object.assign(this.feels[index], res.data)
            })
            .catch(e => {
              console.log(e.response)
            })
        } else {
          this.axios.post('/new', this.editedItem)
            .then(res => {
              this.feels.push(res.data)
            })
            .catch(e => {
              console.log(e.response)
            })
        }
        this.close()
      },
      generarGrafico(key) {
        this.navGrafico = key
        let diaTemp = ''
        let temp = []
        let tempDate = []
        let tempFeel = []

        key!==2 ? [this.ayer=false, this.mespas=false, this.sempas=false] : null

        switch (key) {

          case 1:

            for (let i = this.feels.length-21; i <= this.feels.length-1; i++) {
              let diaPrint = ''
              let mes = this.feels[i].date.substring(5, 7)
              let dia = this.feels[i].date.substring(8, 10)
              let hora = this.feels[i].date.substring(11, 13)
              let min = this.feels[i].date.substring(14, 16)
              diaTemp === dia ? diaPrint = `${hora}:${min}` : [diaPrint = `${dia}/${mes}-${hora}hs`, diaTemp = dia]
              tempDate.push(this.feels[i].feel)
              tempFeel.push(diaPrint)
            }
              this.patriarca.data.datasets[0].data = tempDate
              this.patriarca.data.labels = tempFeel
              this.createChart('canvas-chart', this.patriarca)

            break
          
          case 2:

            for (let i = 0; i < 3; i++) {
              temp = this.feels.filter(element => element.date.substring(8, 10) == new Date(new Date().setDate(new Date().getDate() - i)).getDate())
              this.funfun(0, 23, 11, 13, temp, i)
            }
            break
            
            case 3:
              
              for (let i = 0; i < 3; i++) {
                temp = this.feels.filter(element => element.date.substring(5, 7) == new Date(new Date().setMonth(new Date().getMonth()-i)).getMonth()+1)
                this.funfun(1, 31, 8, 10, temp, i)
              }
              break

            case 4:
              
              for (let i = 0; i < 3; i++) {
                temp = this.feels.filter(element => element.date.substring(5, 7) == new Date(new Date().setMonth(new Date().getMonth()-i)).getMonth()+1)
                this.funfun(1, 12, 5, 7, temp, i)
              }
            break

          default:
          break;
        }
      },
      funfun(start, final, subI, subF, temp, datasetsIndex) {
        let tempDate = []
        let tempFeel = []
        
        for (let i = start; i <= final; i++) {
          let sum = 0
          let cont = 0
          for (let j = 0; j < temp.length; j++) {
            if(i == temp[j].date.substring(subI, subF)) {
              sum += temp[j].feel
              cont++
            }
          }
          let prom = 0
          cont != 0 ? prom = parseInt(sum/cont, 10) : null;
          i < 10 ? i = '0'+i : null;
          tempFeel.push(prom)
          tempDate.push(i)
        }
          this.patriarca.data.datasets[datasetsIndex].data = tempFeel
          this.patriarca.data.labels = tempDate

          this.createChart('canvas-chart', this.patriarca)
      }
    }
  }
</script>