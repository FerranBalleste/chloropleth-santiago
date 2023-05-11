<script setup>
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LGeoJson, LControl } from "@vue-leaflet/vue-leaflet"
import { ref, computed } from 'vue'

import santiago from "./assets/santiago.json"
import listing_counts from "./assets/listing_number.json"

// Random data
let geojson = ref(santiago)

const featured = computed( () => {
  return geojson.value.features.map(i => {
      let c = listing_counts.find( elem => elem.cleansed == i.properties.neighbourhood)
      i.properties.count = c.count
      return i
    }
  )
})

const colors = ["#074cad", "#115f9a", "#1984c5", "#22a7f0", "#48b5c4", "#76c68f", "#a6d75b", "#c9e52f", "#d0ee11", "#d0f400"]
const legend = [
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

function getColor(value) {
  if(value < 10) return colors[9]
  if(value < 20) return colors[8]
  if(value < 30) return colors[7]
  if(value < 40) return colors[6]
  if(value < 50) return colors[5]
  if(value < 100) return colors[4]
  if(value < 500) return colors[3]
  if(value < 1000) return colors[2]
  if(value < 2000) return colors[1]
  else return colors[0]
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
const show = ref(true)
</script>

<template>
  <main>
    <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
      <LTileLayer url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"
                    layer-type="base"
                    name="Stadia Maps Basemap"
      >
      </LTileLayer>
      <LGeoJson
        v-if="show"
        :geojson="featured"
        :optionsStyle="styleFunction"
      />
      <LControl position="bottomright" class="legend">
        <h2>Legend</h2>
        <h3>N. of listings</h3>
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

.legend{
  width: 150px;
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