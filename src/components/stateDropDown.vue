<template>
<div>
  <select v-model="selectedState" v-on:change="handleClick">
      <option v-for="(state,index) in states" :key='index' :value='state'>{{state}}</option>
  </select>
</div>
</template>

<script>
import { eventBus } from '../main.js'
export default {
    name: 'state-drop-down',
    props: ['states'],
    data() {
      return {
        selectedState: null
      }
    },
    methods: {
      handleClick: function() {
        eventBus.$emit('state-selected',this.selectedState)
      }
    }

}
</script>

<style>

</style>