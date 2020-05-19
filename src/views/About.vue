<template>
  <v-container>

  <h1>Verbos:</h1>

  <v-btn class="mr-5 mt-5" v-for="(verb, index) of tareas" :key="index">{{verb}}</v-btn>

  </v-container>
</template>

<script>
import { mapState } from 'vuex'

export default {
  computed: {
    ...mapState(['tareas'])
  }
}
</script>
