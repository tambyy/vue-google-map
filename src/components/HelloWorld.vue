<template>
  <div style="width: 100%; height: 600px">
    <google-map
      style="width: 100%; height: 100%"
      :api-key="apiKey"
      :options="mapOptions"
    >
      <!-- Map Direction need valid API Key -->
      <map-direction
        v-if="origin && destination"
        :options="{
          origin: origin,
          destination: destination,
          waypoints: waypoints,
          polylineOptions: { strokeColor: '#FF000055' },
          suppressMarkers: true,
        }"
      />

      <map-marker
        v-for="marker in markers"
        :key="marker.id"
        :options="{
          position: { lat: marker.lat, lng: marker.lng },
        }"
        @click="
          (e, m) => {
            selectedMarker = m;
            infoWindowContent = getMarkerInfoWindowContent(marker);
          }
        "
      />

      <map-circle
        :options="{
          center: { lat: 50.1773063, lng: 3.2337914 },
          radius: 60000,
          ...circleOptions,
        }"
      />

      <map-info-window
        :options="infoWindowOptions"
        :marker="selectedMarker"
        @closeclick="selectedMarker = null"
      />
    </google-map>
  </div>
</template>

<script>
import GoogleMap from "./google/map/Map.vue";
import MapMarker from "./google/map/Marker.vue";
import MapCircle from "./google/map/Circle.vue";
import MapDirection from "./google/map/Direction.vue";
import MapInfoWindow from "./google/map/InfoWindow.vue";

import { apiKey, mapOptions, circleOptions } from "@/constants/mapSettings";

export default {
  name: "HelloWorld",

  components: {
    GoogleMap,
    MapMarker,
    MapCircle,
    MapDirection,
    MapInfoWindow,
  },

  data() {
    return {
      // You must define your own Google Map API Key
      apiKey,
      mapOptions,
      circleOptions,
      selectedMarker: null,
      infoWindowContent: "",
      markers: [
        { id: 1, name: "A", lat: 50.1773063, lng: 3.2337914 },
        { id: 2, name: "B", lat: 49.848598, lng: 3.2864 },
        { id: 3, name: "C", lat: 49.849998, lng: 2.66667 },
      ],
    };
  },

  methods: {
    getMarkerInfoWindowContent(marker) {
      return marker.name;
    },
  },

  computed: {
    // Direction
    origin() {
      if (this.markers.length < 2) {
        return null;
      }

      const { lat, lng } = this.markers[0];

      return { lat, lng };
    },

    destination() {
      if (this.markers.length < 2) {
        return null;
      }

      const { lat, lng } = this.markers[this.markers.length - 1];

      return { lat, lng };
    },

    waypoints() {
      if (this.markers.length < 2) {
        return null;
      }

      return this.markers
        .slice(1, this.markers.length - 1)
        .map(({ lat, lng }) => ({
          location: { lat, lng },
          stopover: true,
        }));
    },

    // Info window

    infoWindowOptions() {
      return {
        content: this.infoWindowContent,
      };
    },
  },
};
</script>
