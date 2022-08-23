<template>
  <v-app>
    <v-app-bar app color="success" dark>
      <div class="d-flex align-center">
        <v-img alt="Vuetify Logo" class="shrink mr-2" contain src="./assets/trebol.png" transition="scale-transition"
          width="40" />


      </div>

      <v-spacer></v-spacer>

    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
    <v-footer padless>
      <v-col class="text-center" cols="12">
        {{ new Date().getFullYear() }} — <strong>Luis López</strong>
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  data: () => ({
    //
  }),
};
</script>
