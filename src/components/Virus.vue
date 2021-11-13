<template>
    <md-tabs>
        <md-tab md-label="Transmision">
            <Transmision :datos="transmision" :puntos="puntos" @updateAttributesTransmission="updateAttributesTransmission"/>
        </md-tab>
        <md-tab md-label="Sintomas">
            <Sintomas :datos="sintomas" :puntos="puntos" @updateAttributesSymptoms="updateAttributesSymptoms"/>
        </md-tab>
        <md-tab md-label="Habilidad">
            <Habilidad :datos="habilidades" :puntos="puntos" @updateAttributesSkills="updateAttributesSkills"/>
        </md-tab>
    </md-tabs>
</template>

<script>
import Transmision from './Transmision.vue'
import Sintomas from './Sintomas.vue'
import Habilidad from './Habilidad.vue'

export default {
  name: 'Virus',
  props: ['transmision', 'sintomas', 'puntos', 'habilidades'],
  components: {
      Transmision,
      Sintomas,
      Habilidad
  },
  methods: {
    updateAttributesTransmission(e) {
        this.$emit('updateAttributesTransmission', e)
    },
    updateAttributesSymptoms(e) {
        this.$emit('updateAttributesSymptoms', e)
    },
    updateAttributesSkills(e) {
        this.$emit('updateAttributesSkills', e)
    }
  },
}
</script>

<style>

</style>
