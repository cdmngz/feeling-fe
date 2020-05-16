<template>
  <v-container>


    <div class="m-5" v-for="hoy of today" :key="hoy._id">{{hoy.feel}}</div>
    <h2 class="mt-4">Ayer estuviste con emociones entre {{maxFeel}} y {{minHour}}.</h2>
    <h4 class="mt-4">Tu emoción más alta fue a las {{maxHour}}hs, con la descripción {{maxDescrip}}.</h4>

  </v-container>
</template>

<script>

export default {
  data: () => ({
    today: Array
  }),
  created () {
    this.lsFeels()
  },
  computed: {
    maxHour() {
      return this.today[0].date.substring(11, 13)
    },
    maxFeel() {
      return this.today[0].feel
    },
    minHour() {
      return this.today[3].feel
    },
    maxDescrip() {
      return this.today[0].plus
    }
  },
  methods: {
    lsFeels() {
      this.axios.get('/today')
        .then(res => {
          this.today = res.data
        })
        .catch(e => {
          console.log(e.response)
        })
    }
  }
}
</script>