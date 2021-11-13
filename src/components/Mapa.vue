<template>
  <div class="mapa">
    <md-table v-model="datos" md-card @md-selected="onSelect">
      <md-table-row slot="md-table-row" slot-scope="{ item }" md-selectable="single">
        <md-table-cell md-label="Nombre">
          {{ item.country }}
          <img v-if="item.weather == 'Calido'" class="img" src="./../assets/image/wb_sunny_black_24dp.svg" />
          <img v-if="item.weather == 'Frio'" class="img" src="./../assets/image/filter_drama_black_24dp.svg" />
        </md-table-cell>
        <md-table-cell md-label="Poblacion Sana">{{ new Intl.NumberFormat("de-DE").format(item.healthy_population) }}</md-table-cell>
        <md-table-cell md-label="Poblacion Infectada">{{ new Intl.NumberFormat("de-DE").format(item.infected_population) }}</md-table-cell>
        <md-table-cell md-label="Poblacion Muerta">{{ new Intl.NumberFormat("de-DE").format(item.dead_population) }}</md-table-cell>
        <md-table-cell md-label="Puero Maritimo">
          <span v-if="item.transportation_means.maritime == false"> - </span>
          <span v-if="item.transportation_means.maritime == true"> Abierto </span>
          <span v-if="item.transportation_means.maritime == null"> Cerrado </span>
        </md-table-cell>
        <md-table-cell md-label="Aeropuerto">
          <span v-if="item.transportation_means.aerial == false"> - </span>
          <span v-if="item.transportation_means.aerial == true"> Abierto </span>
          <span v-if="item.transportation_means.aerial == null"> Cerrado </span>
        </md-table-cell>
        <md-table-cell md-label="Via Terrestre">          
          <span v-if="item.transportation_means.terrestrial == false"> - </span>
          <span v-if="item.transportation_means.terrestrial == true"> Abierto </span>
          <span v-if="item.transportation_means.terrestrial == null"> Cerrado </span>
        </md-table-cell>
        <md-table-cell md-label="Burbuja">
          <span v-if="item.bubble"> +3 </span>
          <span v-if="!item.bubble"> - </span>
        </md-table-cell>
      </md-table-row>
    </md-table>
  </div>
</template>

<script>

export default {
  name: 'Mapa',
  props: ['datos'],
  data() {
    return {
      selected: false
    }
  },
  methods: {
    onSelect(item) {
      if(!this.selected) {
        this.$emit('startSimulation', item.country);
        this.selected = true;
      }
      console.log(item)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.mapa {
  width: 100%;
  height: 100%;
}


.transporte {
  display: flex;
  margin-top: 20px;
  widows: 100%;
  box-sizing: border-box;
  padding: 10px;
  justify-content: space-between;
  align-items: center;
}

.img {
  margin-left: 5px;
}


</style>
