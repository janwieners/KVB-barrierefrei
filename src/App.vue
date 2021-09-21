<template>
  <barrier-free-map v-bind:opendata="opendata"></barrier-free-map>
</template>

<script language="typescript">
import axios from "axios";

import BarrierFreeMap from "@/components/BarrierFreeMap";

export default {
  name: 'KVB barrierefrei',
  components: {
    BarrierFreeMap
  },
  data() {
    return {
      opendata: 2
    }
  },
  methods: {
    getData() {
      let data = {};
      const vm = this;

      // Data source: https://kvb.koeln/service/open_data.html

      Promise.all([
        axios.get('./data/elevators.json'),
        axios.get('./data/elevatorsdefunc.json'),
        axios.get('./data/escalators.json'),
        axios.get('./data/escalatorsdefunc.json')
          ]).then((result) => {
            data.elevators = result[0].data.features;
            data.elevatorsdefunc = result[1].data.features;
            data.escalators = result[2].data.features;
            data.escalatorsdefunc = result[3].data.features;
            vm.opendata = data;
      });
    }
  },
  mounted() {
    this.getData();
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

* {
  box-sizing: border-box;
}

#app {
  height: 100%;
  width: 100%;
  font-family: Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
