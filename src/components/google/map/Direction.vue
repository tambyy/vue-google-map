<script>
export default {
  inject: ["google", "map"],

  props: {
    directions: {
      type: Object,
      default: null,
    },
    options: {
      type: Object,
      default: () => ({}),
    },
  },

  data() {
    return {
      calculatedDirections: null,
    };
  },

  /**
   * Do not render anything
   */
  render() {
    return null;
  },

  created() {
    this.directionsRenderer = null;
    this.directionsService = null;
  },

  mounted() {
    const { DirectionsRenderer } = this.google.maps;

    this.directionsRenderer = new DirectionsRenderer({
      map: this.map,
      ...this.options,
    });

    if (this.directions) {
      this.calculatedDirections = this.directions;
    } else {
      this.calculateDirections();
    }
  },

  methods: {
    calculateDirections() {
      if (!this.options.origin || !this.options.destination) {
        return;
      }

      if (!this.directionsService) {
        const { DirectionsService } = this.google.maps;

        this.directionsService = new DirectionsService();
      }

      this.directionsService.route(
        {
          origin: this.options.origin,
          destination: this.options.destination,
          waypoints: Array.isArray(this.options.waypoints)
            ? this.options.waypoints
            : [],
          travelMode: this.options.travelMode
            ? this.options.travelMode
            : "DRIVING",
        },
        (response, status) => {
          if (status === "OK") {
            this.calculatedDirections = response;
          } else {
            window.alert("Directions request failed due to " + status);
          }
        }
      );
    },
  },

  beforeUnmount() {
    if (this.directionsRenderer) {
      this.directionsRenderer.setMap(null);
    }
  },

  watch: {
    /**
     * Update calculated directions
     * when directions is changed
     * @param newValue
     */
    directions(newValue) {
      this.calculatedDirections = newValue;
    },

    /**
     * Re-renderer directions
     * when directions is updated
     * @param newValue
     */
    calculatedDirections(newValue) {
      if (!newValue || !this.directionsRenderer) {
        return;
      }

      this.directionsRenderer.setDirections(newValue);
    },

    /**
     * Re-calculate direction
     * when origin is changed
     */
    origin() {
      this.calculateDirections();
    },

    /**
     * Re-calculate direction
     * when destination is changed
     */
    destination() {
      this.calculateDirections();
    },

    /**
     * Re-calculate direction
     * when waypoints is changed
     */
    waypoints() {
      this.calculateDirections();
    },

    /**
     * Update direction options
     * @param newValue
     */
    options(newValue) {
      this.item.setOptions(newValue);
    },
  },
};
</script>
