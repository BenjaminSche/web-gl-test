<template>
  <div class="data-form">
    <v-text-field v-model="nbNodes" label="Nodes" outlined dense />
    <v-text-field v-model="nbLinks" label="Links" outlined dense />
    <v-text-field v-model="nbCircles" label="Circles" outlined dense />
    <v-btn color="primary" @click="generate">Generate</v-btn>
    <v-btn color="primary" @click="play">Play</v-btn>
  </div>
</template>

<script>
import Bus from '@/event/Bus'
export default {
  data() {
    return {
      nbNodes: 10,
      nbLinks: 20,
      nbCircles: 20
    }
  },
  mounted() {
    this.generate()
  },
  methods: {
    generate() {
      let steps = []
      let links = []
      let units = []
      for (let i = 0; i < this.nbNodes; i++) {
        steps.push({
          id: i
        })
      }
      for (let i = 0; i < this.nbLinks; i++) {
        let id_from = steps[this.getRandomNumber(0, steps.length - 1)].id
        let id_to = steps[this.getRandomNumber(0, steps.length - 1)].id
        links.push({
          id: id_from + '|' + id_to,
          from: id_from,
          to: id_to
        })
      }

      for (let i = 0; i < this.nbCircles; i++) {
        units.push({
          id: i,
          linkId: links[this.getRandomNumber(0, links.length - 1)].id,
          progress: Math.random()
        })
      }
      console.log('emit')
      Bus.$emit('datas-generated', {
        steps: steps,
        links: links,
        units: units
      })
    },
    getRandomNumber(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min)
    },
    play() {
      Bus.$emit('stream-play')
    }
  }
}
</script>

<style scoped>
.data-form {
  width: 200px;
  padding: 20px;
}

.v-btn {
  margin-bottom: 20px;
}
</style>
