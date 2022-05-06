<template>
  <div>
  <!--
  <div>
    <ul>
      <li>Latitude (North): {{lat1}}</li>
      <li>Longitude (West): {{lng1}}</li>
      <li>Latitude (South): {{lat2}}</li>
      <li>Longitude (East): {{lng2}}</li>
    </ul>
  </div>
  -->
  <div id="map-wrap" style="height:50vh;width:90vh;margin-left:auto;margin-right:auto">
 <client-only>
   <l-map :zoom=13 :center="center" @update:bounds="showBounds">
     <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></l-tile-layer>
     <!--<l-marker 
          v-for="(marker, key) in markers"
          :key="key"
          :lat-lng="marker.latLng"
          :color="marker.color"
          @click="onClickPlace(marker.uri)"
        >
          <l-popup>
            <a @click="search(marker.uri)">{{ marker.label }}</a>
          </l-popup>
        </l-marker>-->
        <l-circle-marker 
          v-for="(circle, key) in circles"
          :key="key"
          :lat-lng="circle.center"
          :radius="circle.radius"
          :color="circle.color"
          @click="onClickPlace(circle.uri)"
        >
          <l-popup>
            <a @click="search(circle.uri)">{{ circle.label }}</a>
          </l-popup>
        </l-circle-marker>
   </l-map>
 </client-only>
</div>
  <v-container class="container">
    <timeline v-on:click="onClick" v-on:rangechanged="onRangechanged" ref="timeline" :items="items" :groups="groups" :options="options">
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
      //markers: [],
      center: [41.891775,12.486137],
      circles: [],
      statCircles: [],
      groups: [
        {
          id: 0,
          content: 'Roma',
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
      lat1: 0,
      lng1: 0,
      lat2: 0,
      lng2: 0,
    }
  },

  async mounted() {
    const endpoint =
      'https://dydra.com/junjun7613/historical-event-roman/sparql'

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

SELECT * WHERE { 
	?s a <http://www.crm-crm.org/crm-crm/E5_Event> .
    ?s <http://www.example.historical-lod/eventType> ?type.
    ?s <http://www.example.historical-lod/eventCategory> ?category.
    ?s <http://www.example.historical-lod/region> ?region.
    ?s <http://www.w3.org/2000/01/rdf-schema#label> ?eventLabel.
    OPTIONAL{?s <http://www.crm-crm.org/crm-crm/P7_took_place_at> ?place.}
    OPTIONAL{?place <http://www.w3.org/2000/01/rdf-schema#label> ?placeLavel; <http://schema.org/latitude> ?lat; <http://schema.org/longitude> ?lon.}
    ?s <http://www.w3.org/2006/time#hasBeginning> ?beginDate.
    ?s <http://www.w3.org/2006/time#hasEnd> ?endDate.
}`

    const url = `${endpoint}?query=${encodeURIComponent(query)}`

    const data = (await this.$axios.get(url)).data

    console.log(data)
    //mapのマーカー群
    //const markers = []
    const circles = []
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
/*
      if (item.place){
      markers.push({
        uri: item.place,
        label: item.placeLavel,
        latLng: [item.lat, item.lon],
        color: "red",
      })
      continue
      }*/
      if (item.place){
      circles.push({
        uri: item.place,
        label: item.placeLavel,
        center: [item.lat, item.lon],
        radius: 5,
        color: "red",
      })
      continue
      }
    }
    console.log(event_id_map)
    this.event_id_map = event_id_map
    //console.log(markers)
    //this.markers = markers
    this.circles = circles
    this.statCircles = circles

    for (const item of data){

      const ids = items.map(item => item.id)
      //console.log(ids)
      const beginDateId = getIdFromURI(item.beginDate)
      const beginDate = beginDateId.replace("BC","-00")
      const endDateId = getIdFromURI(item.endDate)
      const endDate = endDateId.replace("BC","-00")

      if(ids.includes(item.s)==false && item.beginDate==item.endDate){
      items.push({
          id: item.s,
          group: item.region,
          type: "point",
          start: beginDate,
          end: endDate,
          content: item.eventLabel.value,
        })
        continue
      }else if(ids.includes(item.s)==false && item.beginDate!=item.endDate){
        items.push({
          id: item.s,
          group: item.region,
          start: beginDate,
          end: endDate,
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
      }else{
        ;
      }

      const event_id_map = this.event_id_map
      console.log(event_id_map)

      const newCircles = []
      //const circles = this.statCircles
      const circles = this.circles

      for (const key in event_id_map){
        if(key==event.item){
          console.log(event_id_map[key].place)
          for (const circle of circles){
            if(event_id_map[key].place==circle.uri){
              const newCircle = {
                uri: circle.uri,
                label: circle.lavel,
                center: circle.center,
                radius: 10,
                color: "blue",
              }
              newCircles.push(newCircle)
            }else{
              newCircles.push(circle)
            }
          }
        }else{;}
      }

      this.circles = newCircles

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
    onRangechanged: function(value){
      console.log(value)

      const ST = value.start
      const STTime = ST.getTime()
      console.log(STTime)
      const isoStrST = ST.toISOString()
      console.log(isoStrST)
      

      const ET = value.end
      const ETTime = ET.getTime()
      console.log(ETTime)
      const isoStrET = ET.toISOString()
      console.log(isoStrET)

      //console.log(isoStrST > isoStrET)
      const newItems = []
      const items = this.statItems
      console.log(items)
      for (const key in items){
        //console.log(key)
        const strST = items[key].start
        const strSTDate = Date.parse(strST)
        const strSTTime = strSTDate.valueOf()
        //console.log(strSTTime)
        
        const strET = items[key].end
        const strETDate = Date.parse(strET)
        const strETTime = strETDate.valueOf()
        //console.log(strETTime)
        //console.log(ETDate > strETDate)
        if (STTime < strSTTime && ETTime > strETTime){
          newItems.push(items[key])
        }else{
          ;
          }
      }
      console.log(newItems)
      this.items = newItems
      
      const newItems_eventId = []
      const event_id_map = this.event_id_map
      const newItems_placeId = []

      for (const key in newItems){
        newItems_eventId.push(newItems[key].id)
      }
      console.log(newItems_eventId)

      console.log(event_id_map)
      for (const key of Object.keys(event_id_map)){
        if (newItems_eventId.includes(key)){
          console.log(key)
          newItems_placeId.push(event_id_map[key].place)
        }else{;}
      }

      console.log(newItems_placeId)

      const newCircles = []
      const circles = this.statCircles
      //console.log(circles)
      for (const key in circles){
        if (newItems_placeId.includes(circles[key].uri)){
          newCircles.push(circles[key])
        }else{;}
      }

      console.log(newCircles)
      this.circles = newCircles
    },
    showBounds (bounds) {
      const newItems = []
      const items = this.statItems

      const posNW = bounds.getNorthWest()
      const posSE = bounds.getSouthEast()
      this.lat1 = posNW.lat
      this.lng1 = posNW.lng
      this.lat2 = posSE.lat
      this.lng2 = posSE.lng

      console.log(items)
      for (const key in items) {
        if(items[key].lat<posNW.lat && posSE.lat<items[key].lat && posNW.lng<items[key].lon && items[key].lon<posSE.lng){
          newItems.push(items[key])
        }else{
          ;
        }
      }
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


