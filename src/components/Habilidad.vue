<template>
    <div class="columnasAtributos">
        <div class="columna">
            <span>Nivel 1</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel1" :value="!dato.select" @change="changeCheckBox(index, 1)" :disabled="dato.cost > puntos">{{dato.name}}</md-checkbox>
        </div>
        <div class="columna">
            <span>Nivel 2</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel2" :value="!dato.select" :disabled="!nivel1[index].select || dato.cost > puntos" @change="changeCheckBox(index, 2)"> {{dato.name}}</md-checkbox>
        </div>

    </div>
</template>

<script>

export default {

    name: 'Habilidad',
    props: ['datos', 'puntos'],
    data() {
        return {
           nivel1: this.datos.filter(el => el.level == 1),
           nivel2: this.datos.filter(el => el.level == 2),
        }
    },
    methods: {
        changeCheckBox(index, nivel) {
            let id;
            if(nivel == 1) {
                id = this.nivel1[index].id,
                this.nivel1[index].select = !this.nivel1[index].select;
            } 
            else {
                id = this.nivel2[index].id,
                this.nivel2[index].select = !this.nivel2[index].select;
            } 

            this.$emit('updateAttributesSkills', id)
        }
    }
}
</script>

<style scoped>


.columnasAtributos {
    display: flex;
    width: 50%;
    justify-content: space-between;
    align-items: flex-start;
}

.columna {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}

</style>