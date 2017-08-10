<template>
  <div>
    <div class="google-map" :id="mapName"></div>
    <!-- <button @click="getDistance">Click Me</button> -->
    <li v-for="(branch, index) in markerCoordinates" 
        :key="index">
      {{ branch }}
    </li>
  </div>
</template>

<script>
/* global google */

export default {
  name: 'google-map',
  props: ['name'],
  data () {
    return {
      mapName: this.name + '-map',
      currentPosition: {
        name: 'FlyingComma (here)',
        latitude: 13.801959,
        longitude: 100.575090
      },
      markerCoordinates: [{
        name: 'The Street',
        latitude: 13.770238,
        longitude: 100.572120
      }, {
        name: 'สวนลุมไนท์พลาซ่า',
        latitude: 13.804582,
        longitude: 100.574625
      }, {
        name: 'Central Plaza Ladprao',
        latitude: 13.815836,
        longitude: 100.561391
      }],
      map: null,
      bounds: null,
      markers: []
    }
  },
  computed: {
    branchList () {
      return this.markerCoordinates
    }
  },
  mounted () {
    this.checkGoogle().then(elem => {
      this.initiateMap()
      this.generateDistance()
    })
  },
  methods: {
    checkGoogle () {
      return new Promise((resolve, reject) => {
        let operate = setInterval(() => {
          if (google) {
            resolve(google)
            clearInterval(operate)
          }
        }, 100)
      })
    },
    initiateMap () {
      // Bound Map
      this.bounds = new google.maps.LatLngBounds()
      let element = document.getElementById(this.mapName)
      let mapCenter = this.currentPosition
      this.createMap(element, mapCenter)
      this.addMapMarkder(this.markerCoordinates)
    },
    createMap (element, mapCenter) {
      const options = {
        zoom: 13,
        center: new google.maps.LatLng(mapCenter.latitude, mapCenter.longitude)
      }
      this.map = new google.maps.Map(element, options)
    },
    addMapMarkder (markers, bound) {
      markers.forEach((coord) => {
        const markerPos = new google.maps.LatLng(coord.latitude, coord.longitude)
        const marker = new google.maps.Marker({
          position: markerPos,
          map: this.map
        })
        this.markers.push(marker)
        if (bound) this.map.fitBounds(this.bounds.extend(markerPos))
      })
    },
    generateDistance () {
      this.markerCoordinates.forEach((marker) => {
        marker.distance = this.calculateDistance(this.currentPosition, marker)
      })
      this.sortByDistance()
    },
    rad (x) {
      return x * Math.PI / 180
    },
    calculateDistance (p1, p2) {
      let googleP1 = new google.maps.LatLng(p1.latitude, p1.longitude)
      let googleP2 = new google.maps.LatLng(p2.latitude, p2.longitude)
      let R = 6378137 // Earth’s mean radius in meter
      let dLat = this.rad(googleP2.lat() - googleP1.lat())
      let dLong = this.rad(googleP2.lng() - googleP1.lng())
      let a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(this.rad(googleP1.lat())) * Math.cos(this.rad(googleP2.lat())) *
        Math.sin(dLong / 2) * Math.sin(dLong / 2)
      let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
      let d = R * c
      return (d / 1000).toFixed(2) // returns the distance in kilometer
    },
    sortByDistance () {
      return this.markerCoordinates.sort((a, b) => {
        return a.distance - b.distance
      })
    }
  }
}
</script>

<style scoped>
.google-map {
  width: 800px;
  height: 600px;
  margin: 0 auto;
  background: gray;
}
</style>