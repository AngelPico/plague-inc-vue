<template>
  <div id="app">
    <span class="puntos">Puntos ADN: {{puntos}}</span>
    <span class="fecha">{{fecha.toDateString()}}</span>
    <md-tabs md-alignment="centered">
      <md-tab id="tab-home" md-label="Mapa"><Mapa :datos="paises" @startSimulation="startSimulation" /></md-tab>
      <md-tab id="tab-pages" md-label="Virus">
        <Virus :transmision="transmision" :puntos="puntos" :sintomas="sintomas" :habilidades="habilidades"
          @updateAttributesTransmission="updateAttributesTransmission"
          @updateAttributesSymptoms="updateAttributesSymptoms"
          @updateAttributesSkills="updateAttributesSkills"
        />
      </md-tab>
      <md-tab id="tab-posts" md-label="Paises"><Paises :datos="paises" /></md-tab>
      <md-tab id="tab-favorites" md-label="Cura"><Cura :datos="paisesCura" :porcentaje="porcentajeCura" /></md-tab>
    </md-tabs>
    <md-snackbar v-if="event" :md-duration="1500" :md-active.sync="showSnackbar" md-persistent>
      <span>{{event}}</span>
    </md-snackbar>
  </div>
</template>

<script>
import Mapa from './components/Mapa.vue';
import Virus from './components/Virus.vue';
import Paises from './components/Paises.vue';
import Cura from './components/Cura.vue';
import paisesOrigen from './assets/json/data-pais.json';
import transmision from './assets/json/virus-transmision.json';
import sintomas from './assets/json/virus-sintomas.json';
import habilidades from './assets/json/virus-habilidades.json';

export default {
  name: 'App',
  components: {
    Mapa,
    Virus,
    Paises,
    Cura
  },
  data() {
    return {
      paises: paisesOrigen,
      transmision,
      sintomas,
      habilidades,
      paisesCura: [],
      porcentajeCura: 0,
      puntos: 5,
      event: null,
      fecha: new Date(),
      tasas: [],
      propagacionMaritima: 10,
      propagacionAerea: 10,
      propagacionTerrestre: 20,
      poblacionTotal: 0,
      poblacionSana: 0,
      poblacionEnferma: 0,
      poblacionMuerta: 0,
      tasaCura: 0,
      cura: false,
      propagacionPobres: 0,
      propagacionRicos: 0,
      propagacionCalidos: 0,
      propagacionFrios: 0,
      aumentoTransmision: 0, 
      aumentoGravedad: 0, 
      aumentoLetalidad: 0,
      showSnackbar: false
    }
  },
  created() {

    this.transmision.forEach((element, index) => { element['id'] = index })
    this.sintomas.forEach((element, index) => { element['id'] = index })
    this.habilidades.forEach((element, index) => { element['id'] = index })


    this.paises.forEach(element => {
      if(element.temperature < 20) element['weather'] = 'Frio';
      else if(element.temperature >= 20) element['weather'] = 'Calido';

      if(element.developed && element.support > 0) this.paisesCura.push(element.country);

      this.tasas.push({tasaContagio: 0, aumentoContagio: 0, tasaMuerte: 0, aumentoMuerte: 0});
      this.poblacionTotal += element.total_population;
    });

    this.poblacionSana = this.poblacionTotal;
  },
  methods: {
    startSimulation(e) {

      let index = this.paises.findIndex(el => el.country == e);
      this.paises[index].infected_population += 1;
      this.paises[index].healthy_population -= 1;
      this.paises[index].bubble = true;
      this.puntos += 3;
      this.tasas[index] = {
        tasaContagio: Math.random() * (0.3 - 0.1) + 0.1,
        aumentoContagio: 1.1,
        tasaMuerte: 0,
        aumentoMuerte: 1
      }


      let interval = setInterval(() => {

        this.poblacionEnferma = 0;
        this.poblacionMuerta = 0;
        this.event = null;
        this.showSnackbar = false;
        this.calcularTasa();
        this.actualizarTabla();
        this.sumarDia(); 

        if(this.paises.filter(el => el.infected_population == 0 && el.dead_population != el.total_population).length > 0) this.propagacion();
        if(((this.poblacionEnferma >= this.poblacionTotal * 0.25) 
        || (this.paises.filter(el => (el.infected_population + el.dead_population) == el.total_population).length >= 5))
        && !this.cura) {
          this.cura = true;
          this.tasaCura = 0.1;
          this.showSnackbar = true;
          this.event = "Se esta empezando a desarrollar una cura";
        }

        if(this.cura && this.porcentajeCura < 100) this.calcularCura();
        if(this.porcentajeCura == 100) this.curarPoblacion();
        
        if((this.poblacionSana == 0 && this.poblacionEnferma == 0)) clearInterval(interval);

      }, 2500);

    },
    calcularTasa() {

      this.tasas.forEach(element => {
        if(element.tasaContagio > 0) {
          element.tasaContagio *= element.aumentoContagio;
        } 
        
        if(element.tasaMuerte > 0) {
          element.tasaMuerte *= element.aumentoMuerte;
        }
      });

    },
    actualizarTabla() {

      this.paises.forEach((element, index) => {
        
        if(element.healthy_population >= Math.round(this.tasas[index].tasaContagio) && element.infected_population > 0) {
          element.infected_population += Math.round(this.tasas[index].tasaContagio);
          element.healthy_population = element.total_population - element.infected_population;
        } else if(element.infected_population > 0){
          element.healthy_population = 0;
          element.infected_population = element.total_population;
        }

        if(element.infected_population > element.total_population * 0.45) {
          this.tasas[index].tasaContagio /= this.tasas[index].aumentoContagio;
        }

        if(element.infected_population >= Math.round(this.tasas[index].tasaMuerte)) {
          element.dead_population += Math.round(this.tasas[index].tasaMuerte);
          element.infected_population -= Math.round(this.tasas[index].tasaMuerte); 
        } else if(element.healthy_population == 0) {
          element.infected_population = 0;
          element.dead_population = element.total_population;
        }

        this.poblacionEnferma += element.infected_population;
        this.poblacionMuerta += element.dead_population;

        if(element.infected_population >= element.total_population * 0.6) {
          let aux = this.cerrarMediosDeTransporte(element);
          if(aux != null) {
            element.transportation_means[aux] = null
          }

        }
      })

      this.poblacionSana = this.poblacionTotal - this.poblacionEnferma - this.poblacionMuerta;

    },
    cerrarMediosDeTransporte(pais) {

      let condicion = Math.floor(Math.random() * (11 - 1)) + 1;

      if(condicion == 1) {

         let medio = Math.floor(Math.random() * (4 - 1)) + 1;

         switch (medio) {
            case 1:
              if(pais.transportation_means.maritime == true) return 'maritime';
              break;

            case 2:
              if(pais.transportation_means.aerial == true) return 'aerial';
              break;

            case 3:
              if(pais.transportation_means.terrestrial == true) return 'terrestrial';
              break;
          
            default:
              break;
          }
          
        return null;
      }

    },
    curarPoblacion(){

      let tasaCuracion;

      this.paises.forEach((element, index) => {
        tasaCuracion = Math.floor(Math.random() * ((element.total_population * 0.06) - (element.total_population * 0.03))) + (element.total_population * 0.03);
        if(element.infected_population > tasaCuracion) {
          element.infected_population -= Math.round(tasaCuracion);
          element.healthy_population += Math.round(tasaCuracion);
        } else {
          element.healthy_population += element.infected_population;
          element.infected_population = 0;
        }

        this.tasas[index].tasaContagio = 0;
      })

    },
    sumarDia() {
      let day = this.fecha.getDate() + 1;
      this.fecha = new Date(this.fecha.setDate(day))
    },
    propagacion() {
      
      this.paises.forEach(element => {
        
        if(element.infected_population > 0 && (element.infected_population >= element.total_population * 0.2)) {
          
          let aux = Math.floor(Math.random() * (101 - 1)) + 1;
          
          let via = Math.floor(Math.random() * (4 - 1)) + 1;
          
          switch (via) {
            case 1:
              if(element.transportation_means.maritime == true && aux <= this.propagacionMaritima) this.propagarOtroPais('maritime');
              break;

            case 2:
              if(element.transportation_means.aerial == true && aux <= this.propagacionAerea) this.propagarOtroPais('aerial');
              break;

            case 3:
              if(element.transportation_means.terrestrial == true && aux <= this.propagacionTerrestre) this.propagarOtroPais('terrestrial');
              break;
          
            default:
              break;
          }  

        }

      })

    },
    propagarOtroPais(type) {
      let sanos = this.paises.filter(el => el.healthy_population + el.dead_population == el.total_population && el.transportation_means[type]);
      
      if(sanos.length > 0) {

        let index = Math.floor(Math.random() * (sanos.length - 0)) + 0;
  
        let indexPais = this.paises.findIndex(el => el.country == sanos[index].country);
  
        let aumentoContagio = this.calcularTasaPais(this.paises[indexPais], 1);
        let aumentoMuerte = this.calcularTasaPais(this.paises[indexPais], 2);

        this.paises[indexPais].infected_population += 1;
        this.paises[indexPais].healthy_population -= 1;
        this.paises[indexPais].bubble = true;
        this.puntos += 3;
        this.tasas[indexPais] = {
          tasaContagio: Math.random() * (0.3 - 0.1) + 0.1,
          aumentoContagio: 1.1 + aumentoContagio,
          tasaMuerte: aumentoMuerte > 0 ? Math.random() * (0.5 - 0.1) + 0.1 : 0,
          aumentoMuerte: 1 + aumentoMuerte
        }
  
        if(type == 'maritime') type = 'Maritima';
        if(type == 'aerial') type = 'Aerea';
        if(type == 'terrestrial') type = 'Terrestre';

        this.event = `${sanos[index].country} se contagio via ${type}`;        
        this.showSnackbar = true;
      }

    },
    calcularCura() {

      let support = Math.floor(Math.random() * (101 - 1)) + 1;
      
      if(support <= 5) {

        let desarrollados = this.paises.filter(el => el.developed == true && el.support == 0);
        
        if(desarrollados.length > 0) {

          let index = Math.floor(Math.random() * (desarrollados.length - 0)) + 0;
          let indexPais = this.paises.findIndex(el => el.country == desarrollados[index].country);
          
          this.paises[indexPais].support = 1;
          this.paisesCura.push(this.paises[indexPais].country);
          this.tasaCura *= 1.5;
          this.event = `${this.paises[indexPais].country} se ha unido en la busqueda de una cura`
          this.showSnackbar = true;
        }
      }

      if((this.porcentajeCura + this.tasaCura) > 100) {
        this.porcentajeCura = 100;
      } else {
        this.porcentajeCura += this.tasaCura;
      }

    },    
    calcularTasaPais(pais, opt) {
      let aumento = 0;
      
      if(opt == 1) {
        if(!pais.developed) aumento += this.propagacionPobres;
        if(pais.developed) aumento += this.propagacionRicos;
        if(pais.weather == 'Calido') aumento += this.propagacionCalidos;
        if(pais.weather == 'Frio') aumento += this.propagacionFrios;
        aumento += this.aumentoTransmision;
      } else {
        aumento += this.aumentoLetalidad;
      }

      return aumento;
    },
    updateAttributesTransmission(id) {
      this.transmision[id].select = true;
      this.puntos -= this.transmision[id].cost;
      
      if(this.transmision[id].name == 'Transmision Aire') {
        this.propagacionAerea = this.transmision[id].level == 1 ? 15 : 20;
      }

      if(this.transmision[id].name == 'Transmision Agua') {
        this.propagacionMaritima = this.transmision[id].level == 1 ? 15 : 20;
      }

      if(this.transmision[id].name == 'Transmision Vacunas') {
        this.propagacionPobres = this.transmision[id].level == 1 ? 0.25 : 0.5;

        this.paises.forEach((element, index) => {
          if(!element.developed && element.infected_population > 0) {
            this.tasas[index].aumentoContagio += this.propagacionPobres;
          }
        })

      }

      if(this.transmision[id].name == 'Transmision Insectos') {
        this.propagacionCalidos = this.transmision[id].level == 1 ? 0.25 : 0.5;

        this.paises.forEach((element, index) => {
          if(!element.weather == 'Calido' && element.infected_population > 0) {
            this.tasas[index].aumentoContagio += this.propagacionCalidos;
          }
        })

      }
      
    },
    updateAttributesSymptoms(id) {
      this.sintomas[id].select = true;
      this.puntos -= this.sintomas[id].cost;

      this.aumentoTransmision += this.sintomas[id].infectivity * 0.025;
      this.aumentoLetalidad += this.sintomas[id].lethality * 0.025;

      this.paises.forEach((element, index) => {
        if(element.infected_population > 0) {
          this.tasas[index].aumentoContagio += this.aumentoTransmision;
          this.tasas[index].aumentoMuerte += this.aumentoLetalidad;
          if(this.tasas[index].tasaMuerte == 0) this.tasas[index].tasaMuerte = this.aumentoTransmision > 0 ? Math.random() * (0.5 - 0.1) + 0.1 : 0;
        }
      })
    },
    updateAttributesSkills(id) {
      this.habilidades[id].select = true;
      this.puntos -= this.habilidades[id].cost;

      if(this.habilidades[id].name == 'Resistencia al frio') {
        this.propagacionFrios = this.habilidades[id].level == 1 ? 0.25 : 0.5;

        this.paises.forEach((element, index) => {
          if(!element.weather == 'Frio' && element.infected_population > 0) {
            this.tasas[index].aumentoContagio += this.propagacionFrios;
          }
        })
      }

      if(this.habilidades[id].name == 'Resistencia al calor') {
        this.propagacionCalidos = this.habilidades[id].level == 1 ? 0.25 : 0.5;

        this.paises.forEach((element, index) => {
          if(!element.weather == 'Calido' && element.infected_population > 0) {
            this.tasas[index].aumentoContagio += this.propagacionCalidos;
          }
        })

      }

      if(this.habilidades[id].name == 'Resistencia a los farmacos') {
        this.propagacionRicos = this.habilidades[id].level == 1 ? 0.25 : 0.5;

        this.paises.forEach((element, index) => {
          if(element.developed && element.infected_population > 0) {
            this.tasas[index].aumentoContagio += this.propagacionRicos;
          }
        })

      }

      if(this.habilidades[id].name == 'Endurecimiento genetico') {
        this.tasaCura = this.tasaCura / 1.2;
      }

    },
  },
}
</script>

<style>

#app {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}

.md-content {
  height: auto !important;
}

.md-tab {
  height: 92vh;
  overflow: auto;
}


.puntos {
  position: absolute;
  right: 50px;
  top: 20px;
  z-index: 1;
  font-size: 20px;
  font-weight: 500;
}

.fecha {
  position: absolute;
  left: 50px;
  top: 20px;
  z-index: 1;
  font-size: 20px;
  font-weight: 500;
}

</style>
