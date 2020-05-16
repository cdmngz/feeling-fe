<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="@/assets/Windows.svg"
          transition="scale-transition"
          width="40"
        />

      </div>

      <h3 class="mr-5">Feels</h3>

      <router-link class="white--text mx-2" to="/">Home</router-link> |
      <router-link class="white--text mx-2" to="/about">Verbos</router-link> |
      <router-link class="white--text mx-2" to="/feel">Feels</router-link>

      <v-spacer></v-spacer>

      <v-btn
        href="#"
        target=""
        text
      >
        <span class="mr-2">LogIn</span>
        <v-icon>mdi-heart</v-icon>
      </v-btn>
    </v-app-bar>

    <v-content class="container">
      <router-view/>
    </v-content>
  </v-app>
</template>

<script>

export default {
  name: 'App'
}
</script>
