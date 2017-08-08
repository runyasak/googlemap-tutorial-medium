<template>
  <div>
    <div class="google-map" :id="mapName"></div>
    <button @click="justClick">Click Me</button>
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
    justClick () {
      console.log(this.getLocation())
      this.getLocation().then((res) => {
        console.log(res)
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