<template>
<v-container>

    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(4)">Año</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(3)">Mes</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(2)">Día</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(1)">Últimas 20 anotaciones</v-btn>

  <v-card class="mt-4 mx-auto">
    <v-sheet color="cyan" justify="space-around">
      <v-row v-show="labels.length > 0" justify="space-around">
        <v-switch class="ms-3" v-model="ayer" @click="obtenerGrafico1(1,0,0)" label="Ayer"></v-switch>
        <v-switch class="ms-3" v-model="sempas" @click="obtenerGrafico2(7,0,0)" label="La semana pasada"></v-switch>
        <v-switch class="ms-3" v-model="mespas" @click="obtenerGrafico3(0,1,0)" label="El mes pasado"></v-switch>
      </v-row>
      <v-sparkline
        :value="value"
        :labels="labels"
        color="white"
        height="30"
        label-size="3"
        line-width="0.5"
        padding="10"
        smooth="0"
      ></v-sparkline>
      <v-sparkline
        v-show="ayer"
        :value="value1"
        :labels="labels1"
        color="#00F"
        height="30"
        label-size="3"
        line-width="0.5"
        padding="10"
        smooth="0"
      ></v-sparkline>
      <v-sparkline
        v-show="sempas"
        :value="value2"
        :labels="labels2"
        color="#9F9"
        height="30"
        label-size="3"
        line-width="0.5"
        padding="10"
        smooth="0"
      ></v-sparkline>
      <v-sparkline
        v-show="mespas"
        :value="value3"
        :labels="labels3"
        color="#FF0"
        height="30"
        label-size="3"
        line-width="0.5"
        padding="10"
        smooth="0"
      ></v-sparkline>
    </v-sheet>
  </v-card>

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
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field type="number" min="1" max="10" v-model="editedItem.feel" label="Feel"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.plus" label="Plus"></v-text-field>
                  </v-col>
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
  export default {
    data: () => ({
      ayer: false,
      sempas: false,
      mespas: false,
      aniopas: false,
      feels: [],
      dialog: false,
      grafico: [],
      grafico1: [],
      grafico2: [],
      grafico3: [],
      headers: [
        {
          text: 'Fecha',
          align: 'start',
          sortable: true,
          value: 'date',
        },
        { text: 'Feel', value: 'feel' },
        { text: 'Data adicional', value: 'plus' },
        { text: 'Acciones', value: 'actions', sortable: false },
      ],
      editedIndex: -1,
      editedItem: {},
      defaultItem: {
        feel: 0,
        plus: ''
      }
    }),
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Añadir Feel' : 'Editar Feel'
      },
      value () {
        return this.grafico.map(element => element.feel)
      },
      labels () {
        return this.grafico.map(element => element.date)
      },
      value1 () {
        return this.grafico1.map(element => element.feel)
      },
      labels1 () {
        return this.grafico1.map(element => element.date)
      },
      value2 () {
        return this.grafico2.map(element => element.feel)
      },
      labels2 () {
        return this.grafico2.map(element => element.date)
      },
      value3 () {
        return this.grafico3.map(element => element.feel)
      },
      labels3 () {
        return this.grafico3.map(element => element.date)
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
      lsFeels() {
        this.axios.get('/feel')
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
      obtenerGrafico1(diaRestar, mesRestar, anioRestar) {
        this.grafico1 = []
        let dia = new Date().getDate()-diaRestar
        dia < 1 ? dia+=30 : null
        let mes = new Date().getMonth()-mesRestar+1
        let anio = new Date().getFullYear()-anioRestar
        console.log(dia, mes, anio)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == dia)
            
        for (let i = 0; i < 24; i++) {
          let sum = 0
          let cont = 0
          let prom = 0
          for (let j = 0; j < temp.length; j++) {
            let hora = temp[j].date.substring(11, 13)
            if(i == hora) {
              sum += temp[j].feel
              cont++
            }
          }
          cont != 0 ? prom = parseInt(sum/cont, 10) : null;
          i < 10 ? i = '0'+i : null;
          this.grafico1.push({ date: i, feel: prom })
        }
      },
      obtenerGrafico2(diaRestar, mesRestar, anioRestar) {
        this.grafico2 = []
        let dia = new Date().getDate()-diaRestar
        dia < 1 ? dia+=30 : null
        let mes = new Date().getMonth()-mesRestar+1
        let anio = new Date().getFullYear()-anioRestar
        console.log(dia, mes, anio)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == dia)
            
        for (let i = 0; i < 24; i++) {
          let sum = 0
          let cont = 0
          let prom = 0
          for (let j = 0; j < temp.length; j++) {
            let hora = temp[j].date.substring(11, 13)
            if(i == hora) {
              sum += temp[j].feel
              cont++
            }
          }
          cont != 0 ? prom = parseInt(sum/cont, 10) : null;
          i < 10 ? i = '0'+i : null;
          this.grafico2.push({ date: i, feel: prom })
        }
      },
      obtenerGrafico3(diaRestar, mesRestar, anioRestar) {
        this.grafico3 = []
        let dia = new Date().getDate()-diaRestar
        dia < 1 ? dia+=30 : null
        let mes = new Date().getMonth()-mesRestar+1
        let anio = new Date().getFullYear()-anioRestar
        console.log(dia, mes, anio)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == dia && element.date.substring(5, 7) == mes)
            
        for (let i = 0; i < 24; i++) {
          let sum = 0
          let cont = 0
          let prom = 0
          for (let j = 0; j < temp.length; j++) {
            let hora = temp[j].date.substring(11, 13)
            if(i == hora) {
              sum += temp[j].feel
              cont++
            }
          }
          cont != 0 ? prom = parseInt(sum/cont, 10) : null;
          i < 10 ? i = '0'+i : null;
          this.grafico3.push({ date: i, feel: prom })
        }
      },
      generarGrafico(key) {
        this.grafico = []
        let date = new Date()
        let diaTemp = ''
        let temp = []

        switch (key) {

          case 1:

            for (let i = this.feels.length-21; i <= this.feels.length-1; i++) {
              let diaPrint = ''
              let mes = this.feels[i].date.substring(5, 7)
              let dia = this.feels[i].date.substring(8, 10)
              let hora = this.feels[i].date.substring(11, 13)
              let min = this.feels[i].date.substring(14, 16)
              diaTemp === dia ? diaPrint = `${hora}:${min}` : [diaPrint = `${dia}/${mes}-${hora}:${min}`, diaTemp = dia]
              this.grafico.push({ date: diaPrint, feel: this.feels[i].feel})
            }

            break
          
          case 2:

            temp = this.feels.filter(element => element.date.substring(8, 10) == date.getDate())
            
            for (let i = 0; i < 24; i++) {
              let sum = 0
              let cont = 0
              let prom = 0
              for (let j = 0; j < temp.length; j++) {
                let hora = temp[j].date.substring(11, 13)
                if(i == hora) {
                  sum += temp[j].feel
                  cont++
                }
              }
              cont != 0 ? prom = parseInt(sum/cont, 10) : null;
              i < 10 ? i = '0'+i : null;
              this.grafico.push({ date: i, feel: prom })
            }

            break
            
            case 3:

            temp = this.feels.filter(element => element.date.substring(5, 7) == date.getMonth()+1)

            for (let i = 1; i <= 31; i++) {
              let sum = 0
              let cont = 0
              for (let j = 0; j < temp.length; j++) {
                let dia = temp[j].date.substring(8, 10)
                if(i == dia) {
                  sum += temp[j].feel
                  cont++
                }
              }
              let prom = 0
              cont != 0 ? prom = parseInt(sum/cont, 10) : null
              this.grafico.push({ date: i, feel: prom })
            }
            break

            case 4:

            temp = this.feels.filter(element => element.date.substring(5, 7) == date.getMonth()+1)

            for (let i = 1; i <= 12; i++) {
              let sum = 0
              let cont = 0
              for (let j = 0; j < temp.length; j++) {
                let mes = temp[j].date.substring(5, 7)
                if(i == mes) {
                  sum += temp[j].feel
                  cont++
                }
              }
              let prom = 0
              cont != 0 ? prom = parseInt(sum/cont, 10) : null
              this.grafico.push({ date: i, feel: prom })
            }
            break

          default:
          break;
        }
      }
    }
  }
</script>