<template>
  <div>
    <div class="google-map" :id="mapName"></div>
    <button @click="getDistance">Click Me</button>
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
      markerCoordinates: [{
        latitude: 13.801959,
        longitude: 100.575090
      }, {
        latitude: 13.770238,
        longitude: 100.572120
      }],
      map: null,
      bounds: null,
      markers: []
    }
  },
  mounted () {
    // navigator.geolocation.getCurrentPosition(function (location) {
    //   console.log(location.coords.latitude)
    //   console.log(location.coords.longitude)
    //   console.log(location.coords.accuracy)
    // }, function (err) {
    //   console.log(err)
    // }, {timeout: 10000})
    this.createMap()
  },
  methods: {
    createMap () {
      const operate = setInterval(() => {
        if (google) {
          this.bounds = new google.maps.LatLngBounds()
          const element = document.getElementById(this.mapName)
          // const mapCentre = this.markerCoordinates[0]
          const options = {
            zoom: 14
            // center: new google.maps.LatLng(mapCentre.latitude, mapCentre.longitude)
          }
          this.map = new google.maps.Map(element, options)
          console.log(this.map)
          this.markerCoordinates.forEach((coord) => {
            const markerPos = new google.maps.LatLng(coord.latitude, coord.longitude)
            const marker = new google.maps.Marker({
              position: markerPos,
              map: this.map
            })
            this.markers.push(marker)
            this.map.fitBounds(this.bounds.extend(markerPos))
          })
          clearInterval(operate)
        }
      }, 1000)
    },
    getDistance () {
      let p1 = new google.maps.LatLng(this.markerCoordinates[0].latitude, this.markerCoordinates[0].longitude)
      let p2 = new google.maps.LatLng(this.markerCoordinates[1].latitude, this.markerCoordinates[1].longitude)
      console.log(this.calculateDistance(p1, p2))
    },
    rad (x) {
      return x * Math.PI / 180
    },
    calculateDistance (p1, p2) {
      console.log('p1', p1)
      console.log('p2', p2)
      let R = 6378137 // Earth’s mean radius in meter
      let dLat = this.rad(p2.lat() - p1.lat())
      let dLong = this.rad(p2.lng() - p1.lng())
      let a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(this.rad(p1.lat())) * Math.cos(this.rad(p2.lat())) *
        Math.sin(dLong / 2) * Math.sin(dLong / 2)
      let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
      console.log('C', c)
      let d = R * c
      return (d / 1000).toFixed(2) // returns the distance in kilometer
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