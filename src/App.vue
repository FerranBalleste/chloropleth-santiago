<script setup>
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LGeoJson, LControl } from "@vue-leaflet/vue-leaflet"
import { ref, computed } from 'vue'

import santiago from "./assets/santiago.json"
import listing_counts from "./assets/listing_number.json"

const legendLabels = [
  {"id":1, "alias": "N. of listings"},
  {"id":2, "alias": "XXX"},
  {"id":3, "alias": "YYY"}
]
const selected = ref(legendLabels[0])

// Fill properties values from json
const featured = computed( () => {
  return santiago.features.map(i => {
      let c = listing_counts.find( elem => elem.cleansed == i.properties.neighbourhood)
      i.properties.count = c.count
      return i
    }
  )
})

const colors = ["#074cad", "#115f9a", "#1984c5", "#22a7f0", "#48b5c4", "#76c68f", "#a6d75b", "#c9e52f", "#d0ee11", "#d0f400"]
const legend = computed(() => { 
  switch(selected.value.id) {
    case 1:
      return [
        {"value": 10, "color": colors[9]},
        {"value": 20, "color": colors[8]},
        {"value": 30, "color": colors[7]},
        {"value": 40, "color": colors[6]},
        {"value": 50, "color": colors[5]},
        {"value": 100, "color": colors[4]},
        {"value": 500, "color": colors[3]},
        {"value": 1000, "color": colors[2]},
        {"value": 2000, "color": colors[1]},
        {"value": ">2000", "color": colors[0]}
      ]
    case 2:
      return [
        {"value": 10, "color": colors[9]},
        {"value": 100, "color": colors[8]},
        {"value": 200, "color": colors[7]},
        {"value": 300, "color": colors[6]},
        {"value": 400, "color": colors[5]},
        {"value": 100, "color": colors[4]},
        {"value": 500, "color": colors[3]},
        {"value": 1000, "color": colors[2]},
        {"value": 2000, "color": colors[1]},
        {"value": ">2000", "color": colors[0]}
      ]
    default:
      return undefined
  }
})

function getColor(value) {
  for(const legendItem of legend.value.slice(0,9)){
    if(value < legendItem.value) return legendItem.color
  }
  return legend.value?.[9]?.color
}

function styleFunction(data) {
  const resultStyle = {
    weight: 2,
    color: "#010048",
    opacity: 1,
    fillColor: getColor(data.properties.count),  //send it here
    fillOpacity: .65
  }
  return resultStyle
}

const zoom = ref(10)
const center = ref([-33.42, -70.52])

const options = {
  onEachFeature: function onEachFeature(feature, layer) {
    if (feature.properties.count) {
      layer.bindTooltip(feature.properties.count + " listings");
    } else {
      layer.bindTooltip("Not in port");
    }
  }
}

</script>

<template>
  <main>
    <select v-model="selected" class="selector">
        <option :value="opt" v-for="(opt,index) of legendLabels" :key="index">
          {{ opt.alias }}
        </option>
    </select>

    <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false" :key="selected">
      <LTileLayer name="Stadia Maps Basemap" layer-type="base"
        url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"      
      >
      </LTileLayer>

      <LGeoJson :geojson="featured" :optionsStyle="styleFunction" :options="options"></LGeoJson>

      <LControl position="bottomright" class="legend">
        <h3>Legend</h3>
        <h4>{{ selected.alias }}</h4>
        <div v-for="item of legend" style="display:flex; gap:5px;">
          <div class="color" :style="{backgroundColor: item.color}"></div>
          <span>{{ item.value }}</span>
        </div>
      </LControl>
    </l-map>
  </main>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
}

main {
  height: 100vh;
  width: 100vw;
}

select{
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 500;
  
  width: 10rem;
  padding: 0.5rem 5px;
  border-radius: 5px;
}

select:focus{ 
  outline: 3px solid darkblue;
}

.legend{
  width: 130px;
  margin: 2rem;
  padding: 0.5rem 1rem;
  font-weight: bold;
  border-radius: 5px;
  background-color: whitesmoke;
}

.color{
  width: 40px;
  height: 15px;
}
</style>