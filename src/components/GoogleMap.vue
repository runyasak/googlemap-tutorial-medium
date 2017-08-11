<template>
  <div class="container has-text-centered">
    <div class="columns is-vcentered">
      <div class="column is-5">
        <div class="box related-list">
          <div style="height: 300px; text-align: left;">
            <ul class="menu-list">
              <li v-for="(partner, index) in markerCoordinates" :key="index">
                <a>
                  <span>{{ `${partner.name}` }}</span>
                  <span class="partner-distance">
                    <icon name="location-arrow"></icon> {{`${partner.distance + ' km' || ''}`}}
                  </span>
                </a>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div class="column is-7">
         <div class="google-map" :id="mapName">
        </div> 
      </div>
    </div>
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
        icon: 'https://scontent.fbkk5-6.fna.fbcdn.net/v/t1.0-9/20664138_10209348452935128_8696622600378619713_n.jpg?oh=b42f621ead91ee2f5991e672141dd066&oe=5A381B32',
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
    partnerList () {
      return this.markerCoordinates
    }
  },
  async mounted () {
    await this.checkGoogle()
    this.initiateMap()
    this.generateDistance()
    // this.getCurrentLocation().then(pos => {
    //   console.log(pos)
    // })
  },
  methods: {
    checkGoogle () {
      return new Promise((resolve, reject) => {
        try {
          let operate = setInterval(() => {
            if (google) {
              resolve(google)
              clearInterval(operate)
            }
          }, 100)
        } catch (err) {
          console.log(err)
        }
      })
    },
    initiateMap () {
      // Bound Map
      this.bounds = new google.maps.LatLngBounds()
      let element = document.getElementById(this.mapName)
      this.createMap(element, this.currentPosition)
      this.generateMapMarker(this.markerCoordinates)
    },
    createMap (element, mapCenter) {
      const options = {
        zoom: 13,
        center: new google.maps.LatLng(mapCenter.latitude, mapCenter.longitude)
      }
      this.map = new google.maps.Map(element, options)
      this.generateMapMarker(mapCenter)
    },
    generateMapMarker (markers, bound) {
      if (Array.isArray(markers)) {
        markers.forEach((coord) => {
          let markerPos = new google.maps.LatLng(coord.latitude, coord.longitude)
          this.addMarker(markerPos, null, bound)
        })
      } else {
        let markerPos = new google.maps.LatLng(markers.latitude, markers.longitude)
        let image = 'https://scontent.fbkk5-6.fna.fbcdn.net/v/t1.0-9/20664138_10209348452935128_8696622600378619713_n.jpg?oh=b42f621ead91ee2f5991e672141dd066&oe=5A381B32'
        this.addMarker(markerPos, image, bound)
      }
    },
    addMarker (pos, image, bound) {
      let marker = new google.maps.Marker({
        position: pos,
        icon: image || '',
        map: this.map
      })
      this.markers.push(marker)
      if (bound) this.map.fitBounds(this.bounds.extend(pos))
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
    getCurrentLocation () {
      return new Promise((resolve, reject) => {
        try {
          navigator.geolocation.getCurrentPosition(function (location) {
            console.log(location.coords.latitude)
            console.log(location.coords.longitude)
            console.log(location.coords.accuracy)
            resolve(location.coords)
          })
        } catch (error) {
          reject(new Error(error))
        }
      })
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
  /* width: 450px; */
  height: 440px;
  margin: 0; 
  background: gray;
}
span.partner-distance  {
  display: block;
  font-size: 0.85rem; 
  color: #95A5A6;
}
/* ICON */
.fa-icon {
  width: auto;
  height: 0.85rem;
}
</style>