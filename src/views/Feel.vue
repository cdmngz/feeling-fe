<template>
<v-container>

    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(4)">Año</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(3)">Mes</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(2)">Día</v-btn>
    <v-btn color="primary" dark class="mr-2" @click="generarGrafico(1)">Últimas 20 anotaciones</v-btn>
    
  <v-card class="mt-4 mx-auto">
    <v-sheet color="cyan">
      <v-sparkline
        :value="value"
        :labels="labels"
        label-size="4"
        color="white"
        line-width="1"
        padding="10"
        smooth="1"
      ></v-sparkline>

    </v-sheet>
  </v-card>

  <v-data-table
    :headers="headers"
    :items="feels"
    :items-per-page="15"
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
      feels: [],
      dialog: false,
      grafico: [],
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
      },
    }),
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Añadir Feel' : 'Editar Feel'
      },
      value () {
        let temp = []
        this.grafico.forEach(element => {
          temp.push(element.feel)
        })
        return temp
      },
      labels () {
        let temp = []
        //let diaTemp
        this.grafico.forEach(element => {
          temp.push(element.date)
          //let anio = element.date.substring(2, 4) 
          //let mes = element.date.substring(5, 7)
          //let dia = element.date.substring(8, 10)
          //let hora = element.date.substring(11, 13)
          //let min = element.date.substring(14, 16)
          //mes === '05' ? mes = 'May' : null
          //dia === '10' ? dia = 'Dom' : dia === '11' ? dia = 'Lun' : null
          //diaTemp === dia ? temp.push(hora+':'+min) : [temp.push(dia+'/'+hora), diaTemp = dia]
        })
        return temp
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
        .then(res => {
          console.log(indexDb)
          console.log(res.data)
          this.feels.splice(indexArray, 1)
        })
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
        this.grafico = []
        let date = new Date()
        let temp = []
        let diaTemp = ''

        switch (key) {

          case 1:

            for (let i = this.feels.length-20; i <= this.feels.length-1; i++) {
              let diaNombre = ''
              let diaPrint = ''
              let dia = this.feels[i].date.substring(8, 10)
              let hora = this.feels[i].date.substring(11, 13)
              let min = this.feels[i].date.substring(14, 16)
              dia === '10' ? diaNombre = 'Dom' : dia === '11' ? diaNombre = 'Lun' : null
              diaTemp === dia ? diaPrint = `${hora}:${min}` : [diaPrint = `${diaNombre}-${hora}:${min}`, diaTemp = dia]
              this.grafico.push({ date: diaPrint, feel: this.feels[i].feel})
            }

            break
          
          case 2:

            for (let i = this.feels.length-1; i > 0 ; i--) {
              let dia = this.feels[i].date.substring(8, 10)
              if(dia==date.getDate()) {
                temp.push(this.feels[i])
              }
            }

            temp.reverse()
            
            for (let i = 0; i < 24; i++) {
              let sum = 0
              let cont = 0
              let date = 0
              for (let j = 0; j < temp.length; j++) {
                let hora = temp[j].date.substring(11, 13)
                if(i == hora) {
                  sum += temp[j].feel
                  cont++
                }
              }
              let prom = 0
              cont != 0 ? prom = parseInt(sum/cont, 10) : null
              i<10 ? date = '0'+i : date = i
              this.grafico.push({ date: date, feel: prom })
            }

            break
            
            case 3:

            for (let i = this.feels.length-1; i > 0 ; i--) {
              let mes = this.feels[i].date.substring(5, 7)
              if(mes==date.getMonth()+1) {
                temp.push(this.feels[i])
              } else {
                break;
              }
            }
            temp.reverse()
            
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
            for (let i = this.feels.length-1; i > 0 ; i--) {
              let mes = this.feels[i].date.substring(5, 7)
              if(mes==date.getMonth()+1) {
                temp.push(this.feels[i])
              }
            }
            temp.reverse()
            
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