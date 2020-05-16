<template>
<v-container>

  <h1>Verbos:</h1>

  <v-btn class="mr-5 mt-5" v-for="(verb, index) of verbs" :key="index">{{verb}}</v-btn>
  <v-btn class="mr-5 mt-5" @click="verbs.push('sdf')">+</v-btn>

</v-container>
</template>

<script>
export default {
  name: 'Verbs',
  data: () => ({
    verbs: ['Ejercitarse', 'Programar', 'Amistades', 'Cañas', 'Comida']
  })
}
</script>
