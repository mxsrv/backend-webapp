<template>
  <v-app>
    <v-app-bar
      app
      color="green lighten-1"
      height="70"
    >
      <div class="d-flex align-center">
        <!-- <v-img
          alt="Natur 4.0 Logo"
          class="shrink mr-2"
          contain
          src="https://nature40.github.io/Nature40DocumentationProject/assets/images/official/logo_n40.png"
          transition="scale-transition"
          width="100"
        /> -->

        <v-img
          alt="Vuetify Name"
          class="shrink mr-2"
          contain
          min-width="100"
          src="https://nature40.github.io/Nature40DocumentationProject/assets/images/official/logo_n40.png"
          width="200"
        />
      </div>

      <v-spacer></v-spacer>
      <span
              style="
                font-size: 25px;
                font-family: Arial;
                color: #203864;
                background-color: transparent;
                padding: 10px;
              "
            >
              Backend Web App, connected to: 192.168.0.141
            </span>
      <v-spacer></v-spacer>


      <v-btn
        href="https://www.uni-marburg.de/de/fb19/natur40"
        target="_blank"
        text
      >
        <span class="mr-2" style="color: #203864; ">About Natur 4.0</span>
        <v-icon color="#203864">mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
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
