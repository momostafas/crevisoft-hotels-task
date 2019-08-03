<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Hotels</span>
      </v-toolbar-title>
    </v-app-bar>
    <v-content>
      <Hotels/>
    </v-content>
  </v-app>
</template>

<script>
import Hotels from './components/hotels';

export default {
  name: 'App',
  components: {
    Hotels,
  },
  data: () => ({
    //
  }),
};
</script>
