<template>
  <v-app>
    <v-content>
      <div class="d-flex">
        <Data-Form />
        <Visu-Pixi />
      </div>
    </v-content>
  </v-app>
</template>

<script>
import VisuPixi from '@/components/VisuPixi'
import DataForm from '@/components/DataForm'
export default {
  name: 'App',
  components: {
    VisuPixi,
    DataForm
  },

  data: () => ({
    //
  })
}
</script>
