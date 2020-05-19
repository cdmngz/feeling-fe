<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
      flat
    >
      <h3 class="mx-4">Feels</h3>

      <v-tabs>
        <v-tab to="/">Dashboard</v-tab>
        <v-tab to="/about">Verbos</v-tab>
        <v-tab to="/feel">Feelings</v-tab>
      </v-tabs>

      <v-spacer></v-spacer>
      
    </v-app-bar>

    <v-content class="mt-7">
      <router-view/>
    </v-content>
  </v-app>
</template>

<script>

export default {
  name: 'App'
}
</script>
