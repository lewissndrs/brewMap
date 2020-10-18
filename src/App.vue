<template>
<div id='app'>
  <h1>BREWMAP</h1>
  <button v-on:click="newSearch">New Search</button>
  <state-drop-down :states='statesList' v-if="isStatesList"></state-drop-down>
  <button v-on:click="showSelectedState" v-if="isStatesList">Let's Go!</button>
  <div id='map' style="height: 500px; width: 70%; overflow: auto;">
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
        
        <div v-for="(brewery, index) in filteredList" :key='index'>
          <l-marker :lat-lng='[brewery.latitude,brewery.longitude]' >
          <l-popup>
            <div  @click="linkClick(brewery.website_url)">
            <p>{{ brewery.name }}</p>
            <p><a>{{ brewery.website_url }}</a></p>
            </div>
          </l-popup>
          </l-marker>
        </div>

    </l-map>
  </div>
</div>
</template>

<script>
import {latLng,latLngBounds,getBounds,getCenter} from 'leaflet';
import { LMap, LTileLayer, LMarker, LPopup, LTooltip, LIcon } from 'vue2-leaflet';
import stateDropDown from './components/stateDropDown';
import { eventBus } from './main.js'
export default {
  name: "app",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LTooltip,
    LIcon,
    'state-drop-down': stateDropDown
  },
  data() {
    return {
      bounds: null,

      dataHolder: [],
      statesList: [],

      selectedState: null,

      center: [43.1246149738464, -113.31158377965275],
      zoom: 3,
      mapOptions:{
        zoomSnap: 0.5
      },
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }
  },
  mounted() {
    this.getData();

    eventBus.$on('state-selected',(state) => {this.selectedState = state})
  },
  methods: {
    
    getData: function() {

      const promises = [1, 2, 3, 4, 5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20].map(num => {
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
    },

    getStateData: function() {
      const state = this.selectedState.toLowerCase()
        const promises = [1, 2, 3, 4, 5, 6 , 7 , 8, 9, 10].map(num => {
        return fetch(
          `https://api.openbrewerydb.org/breweries?by_state=${state}&page=${num}&per_page=50`
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
    },

    getStatesList: function() {
      // get list of states
      const stateList = this.filteredList.map((brewery) => brewery.state)
      // reduce to unique states
      this.statesList = stateList.filter((state,index) => stateList.indexOf(state) === index);
    },

    linkClick: function(url) {
      window.open(url,'_blank');
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
    },
    newSearch: function(){
      this.getData();
      this.getBounds();
      this.getStatesList();
    },
    showSelectedState: function(){
      this.getStateData();
      this.getBounds(); // this needs to occur once getStateData has run!!!
    }

  },
  computed: {

    filteredList: function() {
      // filter breweries with no name, coordinates or website
      return this.dataHolder.filter((brewery) => {
        return (brewery.latitude && brewery.longitude && brewery.website_url && brewery.name)
      });
    },


    coOrdList: function() {
      // return list of coordinates
      return this.filteredList.map((brewery) => {
        const lat = parseFloat(brewery.latitude)
        const long = parseFloat(brewery.longitude)
        const result = [lat,long]
        return result;
      })
    },

    isStatesList: function() {
      if (this.statesList[1]) {
        return true;
     } else {
       return false;
     }
    }

    }
  }
</script>

<style>

#app {
  margin: auto;
}

</style>