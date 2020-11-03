<template>
  <v-container>
    <v-row class="text-center">
      <v-col
        class="d-flex"
        cols="12"
        sm="6"
      >
        <v-select
          :items="items"
          label="IPA O NACION"
          v-model="check"
          @change="cambioEstacion"
          outlined
        ></v-select>
      </v-col>
      <v-col
        class="d-flex"
        cols="12"
        sm="6"
        v-if="estacionesActuales != null"
      >
        <v-autocomplete
        v-model="estacionSeleccionada"
        :items="estacionesActuales"
        color="white"
        :item-text="itemText"
        :item-value="itemValue"
        label="Estación"
        @change="eligioEstacion"
        outlined
        filled
        solo
      ></v-autocomplete>
      </v-col>

      <v-col cols="12" v-if="data.length < 1">
        <v-skeleton-loader
        class="mx-auto"
        max-width="100%"
        type="table"
        :loading="true"
        >
        </v-skeleton-loader>
      </v-col>  
      <v-col cols="12" v-else>
        <p class="font-weight-black" v-if="this.check == 'NACION'">
          ULTIMA ACTUALIZACIÓN: {{ date }}
        </p>
        <p class="font-weight-black" v-if="this.check == 'IPA'">
          ULTIMA ACTUALIZACIÓN: {{ data[0].fecha['date'] }}
        </p>
        <v-data-table
          :headers="headers"
          :items="data"
          :items-per-page="5"
          class="elevation-1"
        >
        </v-data-table>
      </v-col>
      
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';
  export default {
    name: 'HelloWorld',

    data: () => ({
      items: ['IPA','NACION'],
      check: '',
      estacionesActuales: null,
      estacionSeleccionada: null,
      headers: [],
      data: [],
      date: null,
      itemText: 'nombre',
      itemValue: 'idEstacion',
    }),
    methods: {
      customFilter (item, queryText) {
        const textOne = item.name.toLowerCase()
        const textTwo = item.abbr.toLowerCase()
        const searchText = queryText.toLowerCase()

        return textOne.indexOf(searchText) > -1 ||
          textTwo.indexOf(searchText) > -1
      },
      async cambioEstacion(e){
        this.estacionesActuales = null
        this.headers = [];
        this.data = [];
        if(e == 'IPA'){
          this.itemText = 'nombre'
          this.itemValue = 'idEstacion'
          this.estacionesActuales = [
          {
            idEstacion: 56,
            nombre: 'Gastre - Salina El Molle'
          },
          {
            idEstacion: 57,
            nombre: 'Estancia Tecka'
          },
          {
            idEstacion: 58,
            nombre: 'Lago Fontana'
          },
          {
            idEstacion: 70,
            nombre: 'Sierra Cuadrada'
          }
          ]
        }
        if(e == 'NACION'){
          this.itemText = 'descripcion';
          this.itemValue = 'idEquipo'
          await axios
          .get('https://ipaback.herokuapp.com/inta/Equipos')
          .then(resp => {
            this.estacionesActuales = resp.data;
          });
        }
      },
      async eligioEstacion(e){
        if (this.check === 'IPA') {
        await axios
          .get('https://ipaback.herokuapp.com/ipa/' + e)
          .then(resp => {
            this.headers = [
              { text: 'ID', value: 'Id' },
              { text: 'Fecha', value: 'fecha[date]' },
              { text: 'LLuvia', value: 'Lluvia' },
              { text: 'Radiación solar', value: 'RadSolar' },
              { text: 'Humedad', value: 'hum' },
              { text: 'Presiòn barometrica', value: 'pBarom' },
              { text: 'Temperatura', value: 'temp' },
              { text: 'Direcciòn del viento', value: 'vDir' },
              { text: 'Velocidad del viento', value: 'vVel' },
            ];
            this.data = resp.data.list;
          });
        }
        if(this.check === 'NACION'){
          await axios
          .get('https://ipaback.herokuapp.com/inta/equipo/' + e)
          .then(resp => {
            this.headers = [
              { text: 'Nombre sensor', value: 'nombreSensor' },
              { text: 'Unidad', value: 'unidad' },
              { text: 'Valor', value: 'valor' },
            ];
            this.data = resp.data.datosSensores;
            this.date = resp.data.fechaUltimaActualizacionDatos;
          });
        }
    }
  }
}
</script>