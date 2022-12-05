<template>
  <div id="map">
  <!--In the following div the HERE Map will render-->
    <div id="mapContainer" style="height:600px;width:100%" ref="hereMap"></div>
  </div>
</template>

<script>
const H = window.H;
var map;

const routingParameters = {
  routingMode: "fast",
  transportMode: "car",
  origin: "52.5160,13.3779", // Brandenburg Gate
  destination: "52.5206,13.3862", // Friedrichstraße Railway Station
  return: "polyline,turnByTurnActions,actions,instructions,travelSummary",
};

export default {
  name: "HereMap",
  props: {
    center: Object
    // center object { lat: 40.730610, lng: -73.935242 }
  },
  data() {
    return {
      platform: null,
      apikey: process.env.VUE_APP_HERE_MAPS_API_KEY
    };
  },
  async mounted() {
    // Initialize the platform object:
    const platform = new window.H.service.Platform({
      apikey: this.apikey
    });
    this.platform = platform;
    this.initializeHereMap();
  },
  methods: {
    initializeHereMap() { // rendering map

      const mapContainer = this.$refs.hereMap;
      // Obtain the default map types from the platform object
      var maptypes = this.platform.createDefaultLayers();

      // Instantiate (and display) a map object:
      map = new H.Map(mapContainer, maptypes.vector.normal.map, {
        zoom: 10,
        center: this.center
        // center object { lat: 40.730610, lng: -73.935242 }
      });

      addEventListener("resize", () => map.getViewPort().resize());

      // add behavior control
      new H.mapevents.Behavior(new H.mapevents.MapEvents(map));

      // add UI
      H.ui.UI.createDefault(map, maptypes);
      // End rendering the initial map

      this.calculateRouteFromAtoB();
    },
    calculateRouteFromAtoB() {
        const router = this.platform.getRoutingService({}, 8);
        router?.calculateRoute(routingParameters, this.onResult, this.onError);
      },
      addRouteShapeToMap(route) {
        route.sections.forEach((section) => {
          // decode LineString from the flexible polyline
          const linestring = H.geo.LineString.fromFlexiblePolyline(
            section.polyline
          );
  
          // Create a polyline to display the route:
          const polyline = new H.map.Polyline(linestring, {
            data: null,
            style: {
              lineWidth: 4,
              strokeColor: "rgba(0, 128, 255, 0.7)",
            },
          });
  
          // Add the polyline to the map
          map.addObject(polyline);  /* This line causes app assets to stay in pending state and causes app crash */
          // And zoom to its bounding rectangle
          map.getViewModel().setLookAtData({
            bounds: polyline.getBoundingBox(),
          });
        });
      },
      onResult(result) {
        const route = result.routes[0];
        // ensure that at least one route was found
        this.addRouteShapeToMap(route);
      },
      onError(error) {
        console.error(error);
      },
  },
};
</script>

<style scoped>
#map {
  width: 60vw;
  min-width: 360px;
  text-align: center;
  margin: 5% auto;
  background-color: #ccc;
}
</style>