<template>
  <v-app>
    <v-row class="red-b">
      <v-col
        cols="12"
        class="px-12 mt-12"
      >
        <v-text-field
          v-model="city"
          label="Search for your city..."
          solo
          clearable
          prepend-inner-icon="mdi-domain"
          append-outer-icon="mdi-map-marker"
        ></v-text-field>
      </v-col>
    </v-row>
  </v-app>
</template>

<script>



export default {
  name: 'App',

  components: {
  },

  data: () => ({
    city: ""
  }),
};
</script>

<style>
  .red-b {
    border: 2px red solid;
  }
  .green-b {
    border: 2px green solid;
  }
</style>

