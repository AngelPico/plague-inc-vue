<template>
    <div class="paisesContent">
        <md-list>
            <md-subheader>Sanos</md-subheader>

            <md-list-item :key="index" v-for="(item, index) in sanos">
                <span>{{item.country}}</span>
            </md-list-item>
        </md-list>
        <md-list>
            <md-subheader>Infectados</md-subheader>

            <md-list-item :key="index" v-for="(item, index) in infectados">
                <span>{{item.country}}</span>
            </md-list-item>
        </md-list>
        <md-list>
            <md-subheader>Destruidos</md-subheader>

            <md-list-item :key="index" v-for="(item, index) in destruidos">
                <span>{{item.country}}</span>
            </md-list-item>
        </md-list>
    </div>
</template>

<script>
export default {
    name: 'Paises',
    props: ['datos'],
    data() {
        return {
            sanos: this.datos,
            infectados: [],
            destruidos: []
        }
    },
    watch: {
        datos: {
            deep: true,
            handler: function (oldValue, newValue) {
                this.sanos = newValue.filter(el => el.total_population == (el.healthy_population + el.dead_population) && el.infected_population == 0);
                this.infectados = newValue.filter(el => el.infected_population > 0);
                this.destruidos = newValue.filter(el => el.total_population == el.dead_population);
            }
        } 
    }
}
</script>

<style scoped>

.paisesContent {
    display: flex;
    justify-content: space-between;
    width: 100%;
    height: 100%;
}

</style>