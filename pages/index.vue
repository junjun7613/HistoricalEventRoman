<template>
  <div><div id="map-wrap" style="height:50vh;width:90vh;margin-left:auto;margin-right:auto">
 <client-only>
   <l-map :zoom=13 :center="[35.025277,135.762222]">
     <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></l-tile-layer>
     <l-marker 
          v-for="(marker, key) in markers"
          :key="key"
          :lat-lng="marker.latLng"
          @click="onClickPlace(marker.uri)"
        >
          <l-popup>
            <a @click="search(marker.uri)">{{ marker.label }}</a>
          </l-popup>
        </l-marker>
   </l-map>
 </client-only>
</div>
  <v-container class="container">
    <timeline v-on:click="onClick" ref="timeline" :items="items" :groups="groups" :options="options">
    </timeline>
  </v-container>
  <div>
    <h3>{{id_content.label}}</h3>
  </div>
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
      statMarkers: [],
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
      statItems: [],
      id_content_map: [],
      id_content: "",
      event_id_map: [],
      options: {
        verticalScroll: true,
      },
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
    const event_id_map = {}
  
    function getIdFromURI(uri){
      const tmp = uri.split('/')
      return tmp[tmp.length - 1]
    }

    for (const item of data) {

      console.log(item)

      const event_id_dict = {}
      event_id_dict.label = item.eventLabel.value
      event_id_dict.place = item.place
      event_id_map[item.s] = event_id_dict

      if (item.place){
      markers.push({
        uri: item.place,
        label: item.placeLavel,
        latLng: [item.lat, item.lon],
      })
      continue
      }
    }
    console.log(event_id_map)
    this.event_id_map = event_id_map
    console.log(markers)
    this.markers = markers
    this.statMarkers

    for (const item of data){

      const ids = items.map(item => item.id)
      //console.log(ids)

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
    this.statItems = items

    const id_content_map = {}

    for (const item of items) {
      //console.log(item)
      const content_dict = {}
      content_dict.label = item.content
      id_content_map[item.id] = content_dict
    }
    console.log(id_content_map)
    this.id_content_map = id_content_map

  },

  methods: {
    onClick: function(event){
      if (event.item!=null){
        console.log(event.item)
        console.log(this.id_content_map[event.item])
        this.id_content = this.id_content_map[event.item]
      }else{;
      }

      const event_id_map = this.event_id_map

    },
    onClickPlace: function(value){
      console.log(value)
      const event_id_map = this.event_id_map
      console.log(event_id_map)
      const newItems = []
      const items = this.statItems

      for (const key in event_id_map){
        if(event_id_map[key].place==value){
          console.log(event_id_map[key].place)
          console.log(key)
          for (const item of items){
            if (key==item.id){
              newItems.push(item)
            }else{;}
          }
        }else{;}
      }

      console.log(newItems)
      this.items = newItems

    },
  },
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


