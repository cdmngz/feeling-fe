<template>
<v-container>

    <v-btn color="primary" class="mr-2" @click="generarGrafico(4)">Año</v-btn>
    <v-btn color="primary" class="mr-2" @click="generarGrafico(3)">Mes</v-btn>
    <v-btn color="primary" class="mr-2" @click="generarGrafico(2)">Día</v-btn>
    <v-btn color="primary" class="mr-2" @click="generarGrafico(1)">Últimas 20 anotaciones</v-btn>

  <v-card class="mt-4 mx-auto">
    <v-sheet color="cyan" justify="space-around">
      <v-row v-show="navGrafico===2" justify="space-around">
        <v-switch class="ms-3" v-model="ayer" @click="obtenerGrafico1()" label="Ayer"></v-switch>
        <v-switch class="ms-3" v-model="sempas" @click="obtenerGrafico2()" label="La semana pasada"></v-switch>
        <v-switch class="ms-3" v-model="mespas" @click="obtenerGrafico3()" label="El mes pasado"></v-switch>
      </v-row>
      <v-sparkline
        :value="value"
        :labels="labels"
        color="white"
        height="40"
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
        height="40"
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
        height="40"
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
        height="40"
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
        <v-toolbar-title @click="funfun(0,24,11,13)">Hay tabla</v-toolbar-title>
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
                    <v-text-field type="number" min="1" max="10" v-model="editedItem.feel" label="Feel" autofocus></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.plus" label="Plus" @keyup.enter="save"></v-text-field>
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
      navGrafico: Number,
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
        { text: 'Descripción', value: 'plus' },
        { text: 'Verbo', array: 'verb' },
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
      obtenerGrafico1() {
        this.grafico1 = []
        let date = new Date()
        date.setDate(date.getDate()-1)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == date.getDate())

        Object.assign(this.grafico1, this.funfun(0, 23, 11, 13, temp))
      },
      obtenerGrafico2() {
        this.grafico2 = []
        let date = new Date()
        date.setDate(date.getDate()-7)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == date.getDate())

        Object.assign(this.grafico2, this.funfun(0, 23, 11, 13, temp))

      },
      obtenerGrafico3() {
        this.grafico3 = []
        let date = new Date()
        date.setMonth(date.getMonth()-1)

        let temp = this.feels.filter(element => element.date.substring(8, 10) == date.getDate() && element.date.substring(5, 7) == date.getMonth())
        
        Object.assign(this.grafico3, this.funfun(0, 23, 11, 13, temp))
        
      },
      generarGrafico(key) {
        this.navGrafico = key
        this.grafico = []
        let diaTemp = ''
        let temp = []

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
              this.grafico.push({ date: diaPrint, feel: this.feels[i].feel})
            }

            break
          
          case 2:

            temp = this.feels.filter(element => element.date.substring(8, 10) == new Date().getDate())
            Object.assign(this.grafico, this.funfun(0, 23, 11, 13, temp))

            break
            
            case 3:
              
            temp = this.feels.filter(element => element.date.substring(5, 7) == new Date().getMonth()+1)
            Object.assign(this.grafico, this.funfun(1, 31, 8, 10, temp))

            break

            case 4:
              
            temp = this.feels.filter(element => element.date.substring(5, 7) == new Date().getMonth()+1)
            Object.assign(this.grafico, this.funfun(1, 12, 5, 7, temp))

            break

          default:
          break;
        }
      },
      funfun(start, final, subI, subF, temp) {
        let jiji = []

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
          jiji.push({ date: i, feel: prom })
        }
        return jiji
      }
    }
  }
</script>