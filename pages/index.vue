<template>
  <div><div id="map-wrap" style="height:50vh;width:90vh;margin-left:auto;margin-right:auto">
 <client-only>
   <l-map :zoom=13 :center="[35.025277,135.762222]">
     <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></l-tile-layer>
     <l-marker
          v-for="(marker, key) in markers"
          :key="key"
          :lat-lng="marker.latLng"
        >
          <l-popup>
            <a @click="search(marker.uri)">{{ marker.label }}</a>
          </l-popup>
        </l-marker>
   </l-map>
 </client-only>
</div>
  <v-container class="container">
    <timeline ref="timeline" :items="items" :groups="groups" :options="options">
    </timeline>
  </v-container>
  </div>
</template>

<script>
import { Timeline } from 'vue-visjs'
export default {
  components: {
    timeline: Timeline,
  },
  data() {
    return {
      markers: [],
      groups: [
        {
          id: 0,
          content: '日本',
        },
        {
          id: 1,
          content: '近畿',
        },{
          id: 2,
          content: '関東',
        },{
          id: 3,
          content: '東北',
        },{
          id: 4,
          content: '中部',
        },
      ],
      items: [],
      options: {},
    }
  },

  async mounted() {
    const endpoint =
      'https://dydra.com/i2k/historical/sparql'

    const query = `PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX ex: <http://www.example.historical-lod/> 
PREFIX event: <http://www.example.historical-lod/event/> 
PREFIX type: <http://www.example.historical-lod/type/> 
PREFIX category: <http://www.example.historical-lod/category/> 
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/> 
PREFIX hcal: <http://datetime.hutime.org/calendar/2.1/date/> 
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX geo: <https://geolod.ex.nii.ac.jp/resource/>
PREFIX schema: <http://schema.org/>

select * where { 
	?s a crm:E5_Event .
    ?s ex:eventType ?type.
    ?s ex:eventCategory ?category.
    ?s ex:region ?region.
    ?s rdfs:label ?eventLabel.
    optional{?s crm:P7_took_place_at ?place.}
    optional{?place rdfs:label ?placeLavel; schema:latitude ?lat; schema:longitude ?lon.}
    ?s time:hasBeginning ?beginDate.
    ?s time:hasEnd ?endDate.
}`

    const url = `${endpoint}?query=${encodeURIComponent(query)}`

    const data = (await this.$axios.get(url)).data

    console.log(data)
    //mapのマーカー群
    const markers = []
    const items = []
  
    function getIdFromURI(uri){
      const tmp = uri.split('/')
      return tmp[tmp.length - 1]
    }

    for (const item of data) {
      if (item.place){
      markers.push({
        uri: item.place,
        label: item.placeLavel,
        latLng: [item.lat, item.lon],
      })
      continue
      }
    }
    console.log(markers)
    this.markers = markers

    for (const item of data){

      const ids = items.map(item => item.id)
      console.log(ids)

      if(ids.includes(item.s)==false && item.beginDate==item.endDate){
      items.push({
          id: item.s,
          group: item.region,
          type: "point",
          start: getIdFromURI(item.beginDate),
          end: getIdFromURI(item.endDate),
          content: item.eventLabel.value,
        })
        continue
      }else if(ids.includes(item.s)==false && item.beginDate!=item.endDate){
        items.push({
          id: item.s,
          group: item.region,
          start: getIdFromURI(item.beginDate),
          end: getIdFromURI(item.endDate),
          content: item.eventLabel.value,
        })
      }
    }
    console.log(items)
    this.items = items
  },
  methods: {},
}
</script>

<style>
.container {
  margin: 0 auto;
  margin-top: 200px;
  min-height: 80vh;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 10px;
  width: 500px;
}
</style>


