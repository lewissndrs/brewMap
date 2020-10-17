<template>
<div id='app'>
  <h1>BREWMAP</h1>
  <div id='map' style="height: 500px; width: 70%; overflow: auto;">
    <button v-on:click="getBounds">BOUNDZ</button>
    <l-map 
      style="height: 80%"
      :zoom ="zoom"
      :bounds="bounds"
      :options="mapOptions"
      :center="center"
      >
        <l-tile-layer
          :url='url'
          :attribution='attribution'
        />

        <l-marker :lat-lng='marker1'/>
        <l-marker :lat-lng='marker2'/>
        
        <div v-for="(point, index) in coOrdList" :key='index'>
          <l-marker :lat-lng='point' />
        </div>

    </l-map>
  </div>
</div>
</template>

<script>
import {latLng,latLngBounds,getBounds,getCenter} from 'leaflet';
import { LMap, LTileLayer, LMarker, LPopup, LTooltip, LIcon } from 'vue2-leaflet';
export default {
  name: "app",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LTooltip,
    LIcon
  },
  data() {
    return {
      marker1: [47.41322, -1.219482],
      marker2: latLng(47.41422, -1.250482),
      bounds: null,
      dataHolder: [],

      center: [47.41322, -1.219482],
      zoom: 13,
      mapOptions:{
        zoomSnap: 0.5
      },
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }
  },
  mounted() {
    this.getData()
  },
  methods: {
    getData: function() {

      const promises = [1, 2, 3, 4, 5].map(num => {
        return fetch(
          `https://api.openbrewerydb.org/breweries?page=${num}&per_page=50`
        ).then(res => res.json());
      });

      Promise.all(promises)
      .then(data => {
        const concatResults = data.reduce((running,toAdd) => {
          return running.concat(toAdd)
        },[])
        return concatResults
      })
      .then(data => this.dataHolder = data)

      // const request = fetch('https://api.openbrewerydb.org/breweries')
      // .then(response => response.json())
      // .then(data => this.dataHolder = data)
    },
    innerClick: function() {
      alert("Click!");
    },
    getBounds: function() {
      // find lowest and highest latitude and longitude


      const lats = this.coOrdList.map((item) => {return item[0]})
      const longs = this.coOrdList.map((item)=> {return item[1]})

      const sortedLats = lats.sort((a,b) => {return a-b})
      const sortedLongs = longs.sort((a,b) => {return a-b})

      const minCoOrd = [sortedLats[0],sortedLongs[0]]
      const maxCoOrd = [sortedLats[sortedLats.length-1],sortedLongs[sortedLats.length-1]]

      console.log('southwest',minCoOrd)
      console.log('northeast',maxCoOrd)

      const bounds = latLngBounds(minCoOrd,maxCoOrd);
      this.bounds = bounds;
      this.center = bounds.getCenter();
    }
  },
  computed: {

    filteredList: function () {
      // filter breweries with no name, coordinates or website
      return this.dataHolder.filter((brewery) => {
        return (brewery.latitude && brewery.longitude && brewery.website_url && brewery.name)
      })
    },
    coOrdList: function() {
      // return list of coordinates
      return this.filteredList.map((brewery) => {
        const lat = parseFloat(brewery.latitude)
        const long = parseFloat(brewery.longitude)
        const result = [lat,long]
        return result;
      })
    }
  }
}
</script>

<style>

#app {
  margin: auto;
}

</style>