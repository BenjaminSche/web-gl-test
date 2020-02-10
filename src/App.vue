<template>
  <v-app>
    <v-content>
      <div class="d-flex">
        <Data-Form />
        <div>
          <div class="flex-row mb-4 mt-4">
            <v-btn text to="/">Pixi.js</v-btn>
            <v-btn text to="/three-js">Three.js</v-btn>
          </div>
          <router-view />
        </div>
      </div>
    </v-content>
  </v-app>
</template>

<script>
import DataForm from '@/components/DataForm'
export default {
  name: 'App',
  components: {
    DataForm
  },

  data: () => ({
    //
  })
}
</script>
