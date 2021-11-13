<template>
    <div class="curaContent">
        <md-list>
            <md-subheader>Paises apoyando</md-subheader>

            <md-list-item :key="index" v-for="(item, index) in datos">
                <span>{{item}}</span>
            </md-list-item>
        </md-list>
        <span class="porcentaje">{{porcentaje.toFixed(0)}}%</span>
        <md-progress-bar md-mode="determinate" :md-value="porcentaje"></md-progress-bar>
    </div> 
</template>


<script>
export default {
    name: 'Cura',
    props: ['datos', 'porcentaje']
}
</script>

<style scoped>
    .curaContent {
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: flex-start;
    }

    .md-list {
        margin-top: 20px;
        max-height: 300px;
        min-height: 150px;
        overflow-y: scroll;
        width: 100%;
    }

    .porcentaje {
        margin-top: 20px;
        width: 50%;
        text-align: center;
        font-size: 20px;
    }

    .md-progress-bar {
        margin-top: 20px;
        width: 50%;
        height: 10px;
    }
</style>