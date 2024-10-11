<template>
  <div>
    <div style="width: 100%; min-height: 100%" ref="googleMap"></div>
    <template v-if="Boolean(this.google) && Boolean(this.map)">
      <slot />
    </template>
  </div>
</template>

<script>
import { computed } from "vue";
import { Loader } from "@googlemaps/js-api-loader";

export default {
  props: {
    options: Object,
    apiKey: String,
  },

  data() {
    return {
      google: "test",
      map: null,
    };
  },

  provide() {
    return {
      google: computed(() => this.google),
      map: computed(() => this.map),
    };
  },

  created() {
    this.marker = null;
  },

  async mounted() {
    const loader = new Loader({
      apiKey: this.apiKey,
    });

    loader.load().then((google) => {
      this.google = google;
      this.initializeMap();
      this.addListeners();
    });
  },

  methods: {
    initializeMap() {
      const mapContainer = this.$refs.googleMap;
      this.map = new this.google.maps.Map(mapContainer, this.options);
    },
    addListeners() {
      this.events.forEach((e) => {
        this.map.addListener(e, (event) => {
          this.$emit("map-" + e, event, this.map, this.google);
        });
      });
    },
    removeListeners() {
      this.events.forEach((e) => {
        this.map.clearListeners(e);
      });
    },
  },

  beforeUnmount() {
    if (this.map) {
      this.removeListeners();
    }
  },

  watch: {
    options(newValue) {
      this.map.setOptions(newValue);
    },
  },

  computed: {
    events() {
      return Object.keys(this.$attrs)
        .filter((attrName) => attrName.startsWith("onMap"))
        .map((attrName) => attrName.slice(5).toLowerCase());
    },
  },
};
</script>
