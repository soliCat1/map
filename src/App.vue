<template>
  <div id="map"></div>
  <div class="information">
    <dl>
      <div class="list-item">
        <dt>Номер поезда:</dt>
        <dd>{{ locoNumber }}</dd>
      </div>
      <div class="list-item">
        <dt>Тип поезда:</dt>
        <dd>{{ locoType }}</dd>
      </div>
    </dl>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import { location } from "@/mocks/location.js";
import { temperatures } from "@/mocks/temperatures.js";

export default {
  data: () => ({
    locoNumber: location.LocoNumber,
    locoType: location.LocoType,
    time: location.Timestamp,
    lat: location.Latitude,
    lng: location.Longitude,
    tempTime: temperatures.Timestamp,
    temp: temperatures.OutsideTemp,
    icon: L.divIcon({
      className: "icon",
      iconSize: [20, 20],
      iconAnchor: [10, 10],
    })
  }),
  mounted() {
    const map = L.map("map").setView([59.13262376, 37.84800256], 15);
    const markerArray = [];
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png").addTo(map)
    this.markers.forEach((marker) => {
      if (marker.lat !== "NA" && marker.lng !== "NA") {
        const newMarker = L.marker([marker.lat, marker.lng], { icon: this.icon }).addTo(map)
        newMarker.bindPopup(`Темепература воздуха за окном: ${marker.temp}`)
        newMarker.on('click', () => newMarker.openPopup())
        markerArray.push([marker.lat, marker.lng])
      }
    })
    const polyline = L.polyline(markerArray, {color: "#57dd4b", weight: 10}).addTo(map)
    map.fitBounds(polyline.getBounds())
    polyline.on('mouseover', (e) => {
      const latlng = e.latlng;
      let closestMarker = null;
      let closestDistance = Infinity;
      this.markers.forEach((marker) => {
        if (marker.lat !== 'NA' && marker.lng !== 'NA') {
          const markerLatLng = L.latLng(marker.lat, marker.lng);
          const distance = latlng.distanceTo(markerLatLng);
          if (distance < closestDistance) {
            closestMarker = marker;
            closestDistance = distance;
          }
        }
      })
      if (closestMarker) {
        const closestMarkerLatLng = L.latLng(
          closestMarker.lat,
          closestMarker.lng
        )
        const closestMarkerPopup = L.popup()
          .setLatLng(closestMarkerLatLng)
          .setContent(`Температура воздуха за окном: ${closestMarker.temp}`)
          .openOn(map);
        console.log(closestMarkerPopup);
      }
    })
    polyline.on('mouseout', () => {
      map.closePopup()
    })
  },
  methods: {
    createTemp(time) {
      for (let i = this.temperatures.length - 1; i >= 0; i--) {
      if (time.getTime() >= this.temperatures[i].time) {
        return this.temperatures[i].temp
      }
    }
  }
},
  computed: {
    markers() {
      const array = [];
      for (let i = 0; i < this.time.length; i++) {
        const obj = {
          locoNumber: this.locoNumber,
          lat: this.lat[i],
          lng: this.lng[i],
          time: new Date(this.time[i]),
          temp: this.createTemp(new Date(this.time[i]))
        }
        array.push(obj)
      }
      return array
    },
    temperatures() {
      const array = [];
      for (let i = 0; i < this.temp.length; i++) {
        const obj = {
          temp: this.temp[i],
          time: new Date(this.tempTime[i]),
        };
        array.push(obj);
      }
      return array;
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

#map {
  width: 100vw;
  height: 100vh;
}

.icon {
  border-radius: 50%;
  background-color: #57dd4b;
  border: 4px solid #ffffff;
}

.information {
  position: absolute;
  bottom: 5%;
  right: 5%;
  z-index: 100000;
  background-color: #4f4444;
  border-radius: 10px;
  padding: 20px;
  width: 250px;
  font-size: 20px;
  color: #1fb912;
  font-weight: 700;
  border: 2px solid rgba(0,0,0,0.2);
  background-clip: padding-box;
}

.list-item {
  display: flex;
  justify-content: space-between;
}

.list-item + .list-item {
  margin-top: 20px;
}

.leaflet-touch .leaflet-bar a {
  background-color: #4f4444;
}

.leaflet-bar a {
  color: #57dd4b;
  border-bottom: none;
}
</style>
