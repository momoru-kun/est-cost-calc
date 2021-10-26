<template>
  <v-app id="app">
    <v-app-bar app dark color="primary">
      <v-app-bar-title> Калькулятор ебать </v-app-bar-title>
    </v-app-bar>
    <v-main>
      <graph-drawer/>
    </v-main>
  </v-app>
</template>

<script>
import GraphDrawer from './components/GraphDrawer.vue'

export default {
  name: 'App',
  components: {
    GraphDrawer
  }
}
</script>

<style>
</style>
