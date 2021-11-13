<template>
    <div class="columnasAtributos">
        <div class="columna">
            <span>Leves</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel1" :value="!dato.select" :disabled="dato.cost > puntos" @change="changeCheckBox(index, 1)">{{dato.name}}</md-checkbox>
        </div>
        <div class="columna">
            <span>Intermedios</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel2" :value="!dato.select" :disabled="dato.cost > puntos" @change="changeCheckBox(index, 2)"> {{dato.name}}</md-checkbox>
        </div>
        <div class="columna">
            <span>Avanzados</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel3" :value="!dato.select" :disabled="dato.cost > puntos" @change="changeCheckBox(index, 3)">{{dato.name}}</md-checkbox>
        </div>
        <div class="columna">
            <span>Criticos</span>
            <md-checkbox :key="index" v-for="(dato, index) in nivel4" :value="!dato.select" :disabled="dato.cost > puntos" @change="changeCheckBox(index, 4)"> {{dato.name}}</md-checkbox>
        </div>

    </div>
</template>

<script>

export default {

    name: 'Sintomas',
    props: ['datos', 'puntos'],
    data() {
        return {
           nivel1: this.datos.filter(el => el.level == 1),
           nivel2: this.datos.filter(el => el.level == 2),
           nivel3: this.datos.filter(el => el.level == 3),
           nivel4: this.datos.filter(el => el.level == 4),
        }
    },
    methods: {
        changeCheckBox(index, nivel) {
            let id
            switch (nivel) {
                case 1:
                    this.nivel1[index].select = !this.nivel1[index].select;
                    id = this.nivel1[index].id;
                    break;

                case 2:
                    this.nivel2[index].select = !this.nivel2[index].select;
                    id = this.nivel2[index].id;
                    break;
                
                case 3:
                    this.nivel3[index].select = !this.nivel3[index].select;
                    id = this.nivel3[index].id;
                    break;
                
                case 4:
                    this.nivel4[index].select = !this.nivel4[index].select;
                    id = this.nivel4[index].id;
                    break;

                default:
                    break;
            }

            this.$emit('updateAttributesSymptoms', id)
        }
    }
}
</script>

<style scoped>


.columnasAtributos {
    display: flex;
    width: 75%;
    justify-content: space-between;
    align-items: flex-start;
}

.columna {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}

</style>