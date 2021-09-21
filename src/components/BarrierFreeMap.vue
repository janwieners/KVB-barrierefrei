<template>
  <div id="mapcontrols">
    <h1>KVB barrierefrei</h1>
    <ul>
      <li>
        <input type="checkbox" id="elevators" value="elevators" @change="toggle($event)"/>
        <label for="elevators">Aufzüge: {{ opendata.elevators?.length }}</label></li>
      <li>
        <input type="checkbox" id="elevatorsdefunc" value="elevatorsdefunc" checked @change="toggle($event)"/>
        <label for="elevatorsdefunc">Aufzugsstörungen: {{ opendata.elevatorsdefunc?.length }}
          ({{ (opendata.elevatorsdefunc?.length / opendata.elevators?.length * 100).toFixed(2) }}%)</label>
      </li>
      <li>
        <input type="checkbox" id="escalators" value="escalators" @change="toggle($event)"/>
        <label for="escalators">Fahrtreppen: {{ opendata.escalators?.length }}</label>
      </li>
      <li>
        <input type="checkbox" id="escalatorsdefunc" value="escalatorsdefunc" checked @change="toggle($event)"/>
        <label for="escalatorsdefunc">Fahrtreppenstörungen: {{ opendata.escalatorsdefunc?.length }}
          ({{ (opendata.escalatorsdefunc?.length / opendata.escalators?.length * 100).toFixed(2) }}%)</label>
      </li>
      <li><small>Quelle: <a href="https://kvb.koeln/service/open_data.html">Offene Daten KVB</a></small></li>
    </ul>
  </div>
  <div id="map"></div>
</template>

<script lang="typescript">
import L from "leaflet";
import "leaflet/dist/leaflet.css";

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {
  name: 'BarrierFreeMap',
  props: ['opendata'],
  data() {
    return {
      map: null,
      elevators: [],
      elevatorsdefunc: [],
      escalators: [],
      escalatorsdefunc: []
    }
  },
  watch: {
    opendata: {
      handler: function () {
        this.initMap();
      }
    }
  },
  methods: {
    initMap: function () {
      this.map = L.map("map");
      L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'Map data &copy; <a href="http://openstreetmap.org/copyright">OpenStreetMap</a> contributors</a>'
      }).addTo(this.map);

      this.map.attributionControl
          .setPrefix('')
          .addAttribution(
              'Powered by <a href="https://www.geoapify.com/" target="_blank">Geoapify</a> | © OpenStreetMap <a href="https://www.openstreetthis.map.org/copyright" target="_blank">contributors</a>'
          );

      this.map.invalidateSize();
      this.map.setView([50.9412784,6.9582814], 15);
      //this.map.locate({setView: true, maxZoom: 16, watch: false});
      L.control.scale().addTo(this.map);

      let redIcon = new L.Icon({
        iconRetinaUrl: require('../assets/marker-icon-2x-red.png'),
        iconUrl: require('../assets/marker-icon-2x-red.png'),
        shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [25, 20]
      });

      let elevators = [];
      for (let elevator of this.opendata.elevators) {
        elevators.push(
            L.marker([elevator.geometry.coordinates[1], elevator.geometry.coordinates[0]])
                .bindPopup(`<h2>Aufzug</h2>${elevator.properties.Bezeichnung}`));
      }
      this.elevators = L.layerGroup(elevators);

      let elevatorsdefunc = [];
      for (let elevator of this.opendata.elevatorsdefunc) {
        elevatorsdefunc.push(
            L.marker([elevator.geometry.coordinates[1], elevator.geometry.coordinates[0]], {icon: redIcon})
                .bindPopup(`<h2>Aufzugsstörung</h2>${elevator.properties.Bezeichnung}<br>Stand: ${elevator.properties.timestamp}`));
      }
      this.elevatorsdefunc = L.layerGroup(elevatorsdefunc).addTo(this.map);

      let escalators = [];
      for (let escalator of this.opendata.escalators) {
        escalators.push(
            L.marker([escalator.geometry.coordinates[1], escalator.geometry.coordinates[0]])
                .bindPopup(`<h2>Fahrtreppe</h2>${escalator.properties.Bezeichnung}<br>`));
      }
      this.escalators = L.layerGroup(escalators);

      let escalatorsdefunc = [];
      for (let escalator of this.opendata.escalatorsdefunc) {
        escalatorsdefunc.push(
            L.marker([escalator.geometry.coordinates[1], escalator.geometry.coordinates[0]], {icon: redIcon})
                .bindPopup(`<h2>Fahrtreppenstörung</h2>${escalator.properties.Bezeichnung}<br>Stand: ${escalator.properties.timestamp}`));
      }
      this.escalatorsdefunc = L.layerGroup(escalatorsdefunc).addTo(this.map);
    },
    toggle(layer) {
      switch (layer.target.defaultValue) {
        case 'elevators':
          layer.target.checked ? this.map.addLayer(this.elevators) : this.map.removeLayer(this.elevators);
          break;
        case 'elevatorsdefunc':
          layer.target.checked ? this.map.addLayer(this.elevatorsdefunc) : this.map.removeLayer(this.elevatorsdefunc);
          break;
        case 'escalators':
          layer.target.checked ? this.map.addLayer(this.escalators) : this.map.removeLayer(this.escalators);
          break;
        case 'escalatorsdefunc':
          layer.target.checked ? this.map.addLayer(this.escalatorsdefunc) : this.map.removeLayer(this.escalatorsdefunc);
          break;
      }
    },
  }
};
</script>

<style scoped>
#mapcontrols {
  position: absolute;
  top: 0;
  right: 0;
  width: 19rem;
  z-index: 2;
  background-color: rgba(255, 255, 255, 0.9);
  font-size: .9em;
  line-height: 1.5em;
  padding: 1rem;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

ul li {
  margin: .5rem 0;
}

#map {
  z-index: 1;
  width: 100%;
  height: 100%;
}

h1 {
  font-weight: lighter;
}

#description {
  padding: 0 2rem 1rem;
  text-align: justify;
}

small {
  display: block;
  margin-top: .3rem;
}
</style>
