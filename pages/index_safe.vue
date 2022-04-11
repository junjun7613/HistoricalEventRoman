<template>
  <div><div id="map-wrap" style="height:50vh;width:90vh;margin-left:auto;margin-right:auto">
 <client-only>
   <l-map :zoom=13 :center="[35.025277,135.762222]">
     <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></l-tile-layer>
     <l-marker :lat-lng="[35.025277,135.762222]"><l-popup>
       <h3>御所</h3>
       </l-popup></l-marker>
     <l-marker :lat-lng="[34.935277, 135.761388]"><l-popup>
       <h3>伏見</h3>
       </l-popup></l-marker>
     <l-marker :lat-lng="[34.932777, 135.745555]"><l-popup>
       <h3>下鳥羽</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[34.964389, 135.743291]"><l-popup>
       <h3>上鳥羽村</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[34.687586, 135.193104]"><l-popup>
       <h3>居留地</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[36.079909, 138.084916]"><l-popup>
       <h3>下諏訪町</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[35.685, 139.754444]"><l-popup>
       <h3>江戸館址</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[38.26776, 140.869539]"><l-popup>
       <h3>仙台市</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[35.715558, 139.770174]"><l-popup>
       <h3>上野公園</h3>
       </l-popup>
       </l-marker>
       <l-marker :lat-lng="[37.487222, 139.93]"><l-popup>
       <h3>会津山</h3>
       </l-popup>
       </l-marker>
     <l-circle
      :lat-lng="[45.758866, 4.8194815]"
      :radius="2000"
      :color="'red'"
     ><l-popup>
       <h3>Lyon</h3>
       </l-popup></l-circle>
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
      items: [
        {
          id: 0,
          group: 1,
          start: "1868-01-27",
          end: "1868-01-30",
          content: '鳥羽・伏見の戦い',
        },
        {
          id: 1,
          group: 0,
          start: "1868-01-27",
          end: "1869-06-27",
          content: '戊辰戦争',
        },
        {
          id: 2,
          group: 0,
          type: "point",
          start: "1868-01-25",
          end: "1868-01-25",
          content: '徳川慶喜征討令',
        },
        {
          id: 3,
          group: 1,
          type: "point",
          start: "1868-02-04",
          end: "1868-02-04",
          content: '神戸事件',
        },
        {
          id: 4,
          group: 0,
          start: "1868-01-25",
          end: "1868-03-24",
          content: '三職八局の制',
        },
        {
          id: 5,
          group: 4,
          type: "point",
          start: "1868-03-26",
          end: "1868-03-26",
          content: '相楽総三ら処刑',
        },
        {
          id: 6,
          group: 0,
          type: "point",
          start: "1868-04-06",
          end: "1868-04-06",
          content: '五箇条の誓文',
        },
        {
          id: 7,
          group: 0,
          type: "point",
          start: "1868-04-07",
          end: "1868-04-07",
          content: '五榜の掲示',
        },
        {
          id: 8,
          group: 2,
          start: "1868-03-24",
          end: "1868-05-03",
          content: '江戸開城',
        },
        {
          id: 9,
          group: 0,
          type: "point",
          start: "1868-06-11",
          end: "1868-06-11",
          content: '政体書公布',
        },
        {
          id: 10,
          group: 3,
          start: "1868-03-24",
          end: "1868-10-16",
          content: '奥羽列藩同盟',
        },
        {
          id: 11,
          group: 3,
          type: "point",
          start: "1868-06-22",
          end: "1868-06-22",
          content: '奥羽列藩同盟成立',
        },
        {
          id: 12,
          group: 2,
          type: "point",
          start: "1868-07-04",
          end: "1868-07-04",
          content: '彰義隊壊滅',
        },
        {
          id: 13,
          group: 0,
          type: "point",
          start: "1868-09-03",
          end: "1868-09-03",
          content: '東京改称',
        },
        {
          id: 14,
          group: 3,
          start: "1868-09-16",
          end: "1868-11-14",
          content: '会津攻撃',
        },
      ],
      options: {},
    }
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
