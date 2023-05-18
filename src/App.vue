<script setup>
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LGeoJson, LControl, LCircle, LTooltip, LPopup } from "@vue-leaflet/vue-leaflet"
import { ref, computed } from 'vue'

import santiago from "./assets/santiago.json"
import listing_counts from "./assets/listing_number2.json"

const legendLabels = [
  {"id":1, "alias": "N. of listings"},
  {"id":2, "alias": "Average Price (Chilean peso)"},
  {"id":3, "alias": "Average Rating"}
]
const selected = ref(legendLabels[0])

// Fill properties values from json
const featured = computed( () => {
  return santiago.features.map(i => {
      let c = listing_counts.find( elem => elem.cleansed == i.properties.neighbourhood)
      if(c){
        i.properties.count = c.count
        i.properties.price = c.price
        i.properties.rating = c.rating
      }
      return i
    }
  )
})

const colors = ["#074cad", "#115f9a", "#1984c5", "#22a7f0", "#48b5c4", "#76c68f", "#a6d75b", "#c9e52f", "#d0ee11", "#d0f400"]
const colors2 = ["#fff1f2", "#ffe4e6", "#fecdd3", "#fda4af", "#fb7185", "#f43f5e", "#e11d48", "#be123c", "#9f1239", "#881337"]
const colors3 = ["#fff7ed", "#ffedd5", "#fed7aa", "#fdba74", "#fb923c", "#f97316", "#ea580c", "#c2410c", "#9a3412", "#7c2d12"]
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
        {"value": 10000, "color": colors2[0]},
        {"value": 20000, "color": colors2[1]},
        {"value": 30000, "color": colors2[2]},
        {"value": 40000, "color": colors2[3]},
        {"value": 50000, "color": colors2[4]},
        {"value": 60000, "color": colors2[5]},
        {"value": 70000, "color": colors2[6]},
        {"value": 80000, "color": colors2[7]},
        {"value": 90000, "color": colors2[8]},
        {"value": ">100000", "color": colors2[9]}
      ]
    case 3:
      return [
        {"value": 0, "color": colors3[0]},
        {"value": 2, "color": colors3[1]},
        {"value": 3, "color": colors3[2]},
        {"value": 4, "color": colors3[3]},
        {"value": 4.2, "color": colors3[4]},
        {"value": 4.4, "color": colors3[5]},
        {"value": 4.6, "color": colors3[6]},
        {"value": 4.8, "color": colors3[7]},
        {"value": 4.9, "color": colors3[8]},
        {"value": ">4.9", "color": colors3[9]}
      ]
    default:
      return undefined
  }
})

function getColor(value) {
  for(const legendItem of legend.value.slice(0,9)){
    if(!value) return "#fff"
    if(value < legendItem.value) return legendItem.color
  }
  return legend.value?.[9]?.color
}

function styleFunction(data) {
  let property = 0
  if(selected.value.id == 1) property = data.properties.count
  else if(selected.value.id == 2) property = data.properties.price
  else if(selected.value.id == 3) property = data.properties.rating
  else property = 0
  const resultStyle = {
    weight: 2,
    color: "#010048",
    opacity: 1,
    fillColor: getColor(property),  //send it here
    fillOpacity: .65
  }
  return resultStyle
}

const zoom = ref(10)
const center = ref([-33.42, -70.52])

function popupHTML(properties){
  return `<div class="popup">
  <h3>${properties.neighbourhood}</h3>
  <p>Count: ${properties.count} listings</p>
  <p>AvgPrice: $${properties.price}</p>
  <p>AvgRating: ${properties.rating}
  <svg height="10" width="10" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 47.94 47.94" xml:space="preserve">
  <path style="fill:#ED8A19;" d="M26.285,2.486l5.407,10.956c0.376,0.762,1.103,1.29,1.944,1.412l12.091,1.757
	c2.118,0.308,2.963,2.91,1.431,4.403l-8.749,8.528c-0.608,0.593-0.886,1.448-0.742,2.285l2.065,12.042
	c0.362,2.109-1.852,3.717-3.746,2.722l-10.814-5.685c-0.752-0.395-1.651-0.395-2.403,0l-10.814,5.685
	c-1.894,0.996-4.108-0.613-3.746-2.722l2.065-12.042c0.144-0.837-0.134-1.692-0.742-2.285l-8.749-8.528
	c-1.532-1.494-0.687-4.096,1.431-4.403l12.091-1.757c0.841-0.122,1.568-0.65,1.944-1.412l5.407-10.956
	C22.602,0.567,25.338,0.567,26.285,2.486z"/>
  </svg>
  </p>
  </div>`
}

const options = computed(() => {
  return {
    onEachFeature: function onEachFeature(feature, layer) {
      if(feature.properties.neighbourhood){
        layer.bindPopup(popupHTML(feature.properties))
      }
      
      if (selected.value.id == 1 && feature.properties.count) {
        layer.bindTooltip(feature.properties.neighbourhood + ": " + feature.properties.count + " listings");
      } 
      else if (selected.value.id == 2 && feature.properties.count) {
        layer.bindTooltip(feature.properties.neighbourhood + ": " + "$" + feature.properties.price + " average");
      }
      else if (selected.value.id == 3 && feature.properties.count) {
        layer.bindTooltip(feature.properties.neighbourhood + ": " + feature.properties.rating + "  stars");
      } else {
        layer.bindTooltip(feature.properties.neighbourhood + ": " +  "No data");
      }
    }
  }
})

const showBubbles = ref(false)
</script>

<template>
  <main>
    <select v-model="selected" class="selector">
        <option :value="opt" v-for="(opt,index) of legendLabels" :key="index">
          {{ opt.alias }}
        </option>
    </select>
    <div class="checkwrap">
      <input v-model="showBubbles" type="checkbox" id="scales" name="scales" checked>
      <label for="scales">Show Bubbles</label>
    </div>

    <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false" :key="selected">
      <LTileLayer name="Stadia Maps Basemap" layer-type="base"
        url="https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png"      
      >
      </LTileLayer>

      <LGeoJson :visible="!showBubbles" :geojson="featured" :optionsStyle="styleFunction" :options="options"></LGeoJson>

      <template v-for="circle in listing_counts" :index="circle.cleansed">
        <LCircle
          :visible="showBubbles && selected.id == 1"
          :lat-lng="[circle.latitude, circle.longitude]"
          :radius="circle.count * 2"
          color="#fb923c"
        >
        <l-tooltip>{{ circle.cleansed }}: {{ circle.count }}</l-tooltip>
      </LCircle>
      <LCircle
          :visible="showBubbles && selected.id == 2"
          :lat-lng="[circle.latitude, circle.longitude]"
          :radius="circle.price/40"
          color="#a855f7"
        >
        <l-tooltip>{{ circle.cleansed }}: ${{ circle.price }}</l-tooltip>
      </LCircle>
      <LCircle
          :visible="showBubbles && selected.id == 3"
          :lat-lng="[circle.latitude, circle.longitude]"
          :radius="circle.rating * 100"
          color="#10b981"
        >
        <l-tooltip>{{ circle.cleansed }}: {{ circle.rating }}</l-tooltip>
      </LCircle>
      </template>

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

.checkwrap{
  background: whitesmoke;
  position: absolute;
  top: 50px;
  right: 10px;
  z-index: 500;
  padding: 0.5rem 1rem;
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