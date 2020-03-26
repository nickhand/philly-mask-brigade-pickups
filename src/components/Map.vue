<template>
  <div>
    <div class="map-header w-100 d-flex justify-content-center align-items-center">
      <div class="legend-entry d-flex flex-row align-items-center justify-content-center">
        <span class="legend-circle legend-circle-active"></span>
        <div class="legend-label ml-1">Active Pickups</div>
      </div>
      <div class="legend-entry d-flex flex-row align-items-center justify-content-center ml-3">
        <span class="legend-circle"></span>
        <div class="legend-label ml-1">Completed Pickups</div>
      </div>
      <div class="legend-entry d-flex flex-row align-items-center justify-content-center ml-3">
        <span class="legend-circle legend-circle-failed"></span>
        <div class="legend-label ml-1">Failed Pickups</div>
      </div>
    </div>
    <MglMap
      :accessToken="accessToken"
      :mapStyle="mapStyle"
      :zoom="zoom"
      :center="center"
      :minZoom="minZoom"
      :maxZoom="maxZoom"
      :scrollZoom="scrollZoom"
      ref="pickupsMap"
    >
      <MglNavigationControl position="top-right" />
      <MglGeolocateControl position="top-right" />

      <MglGeojsonLayer
        :sourceId="cityLimitsSource.data.id"
        :source="cityLimitsSource"
        layerId="cityLimits"
        :layer="cityLimitsLayer"
      />
      <MglGeojsonLayer
        :sourceId="pickupsSource.data.id"
        :source="pickupsSource"
        layerId="pickups"
        :layer="pickupsLayer"
        @mouseenter="showPopup"
        @mouseleave="hidePopup"
      />
    </MglMap>
  </div>
</template>

<script>
import Mapbox from "mapbox-gl";
import {
  MglMap,
  MglNavigationControl,
  MglGeolocateControl,
  MglGeojsonLayer
} from "vue-mapbox";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  components: {
    MglMap,
    MglNavigationControl,
    MglGeolocateControl,
    MglGeojsonLayer
  },
  props: ["pickupFeatures"],
  data() {
    return {
      accessToken:
        "pk.eyJ1IjoibmljaG9sYXNoYW5kIiwiYSI6ImNrODJoeDN1dzEyYXUzZGxucGlpZHJuMjcifQ.0g3_RJUyHZQWn6-pMUgtJw", // your access token. Needed if you using Mapbox maps
      mapStyle: "mapbox://styles/mapbox/dark-v10", // your map style
      zoom: 10,
      minZoom: 10,
      maxZoom: 15,
      center: [-75.15, 39.99],
      popup: null,
      pickupsLayer: {
        type: "circle",
        paint: {
          "circle-color": [
            "match",
            ["get", "Success?"],
            "Done",
            "rgba(192, 108, 132, 0.5)",
            "Failed",
            "rgba(207, 207, 207, 0.5)",
            "rgba(37, 206, 247, 0.5)"
          ],
          "circle-stroke-width": 1,
          "circle-stroke-color": [
            "match",
            ["get", "Success?"],
            "Done",
            "rgba(192, 108, 132, 1)",
            "Failed",
            "rgba(207, 207, 207, 1)",
            "rgba(37, 206, 247, 1)"
          ],
          "circle-radius": [
            "match",
            ["get", "Success?"],
            "Done",
            6,
            "Failed",
            6,
            12
          ]
        }
      },
      cityLimitsSource: {
        type: "geojson",
        data: {
          id: "cityLimitsSource",
          type: "FeatureCollection",
          features: require("@/data/City_Limits.json")["features"]
        }
      },
      cityLimitsLayer: {
        type: "line",
        paint: {
          "line-color": "#fff",
          "line-width": 2
        }
      }
    };
  },
  computed: {
    scrollZoom() {
      if (this.getWidth() > 768) return true;
      else return false;
    },
    pickupsSource() {
      return {
        type: "geojson",
        data: {
          id: "pickupsSource",
          type: "FeatureCollection",
          features: this.pickupFeatures
        }
      };
    }
  },

  created() {
    // We need to set mapbox-gl library here in order to use it in template
    this.mapbox = Mapbox;

    // Create a popup, but don't add it to the map yet.
    this.popup = new Mapbox.Popup({
      closeButton: false,
      closeOnClick: false,
      offset: -30
    });
  },
  methods: {
    getWidth() {
      return window.screen.height;
    },
    showPopup(e) {
      let map = e.map;
      map.getCanvas().style.cursor = "pointer";

      let coordinates = e.mapboxEvent.features[0].geometry.coordinates.slice();
      let address =
        e.mapboxEvent.features[0].properties[
          "Street Name & Number (i.e. 1234 Broad St)"
        ];
      let masks =
        e.mapboxEvent.features[0].properties[
          "Approximate number of unused masks"
        ];

      let description = `<div class='my-tooltip'><div class='tooltip-title'>${address}</div><div class='tooltip-line'>${masks} masks</div></div>`;
      // Populate the popup and set its coordinates
      // based on the feature found.
      this.popup
        .setLngLat(coordinates)
        .setHTML(description)
        .addTo(map);
    },
    hidePopup(e) {
      let map = e.map;
      map.getCanvas().style.cursor = "";
      this.popup.remove();
    }
  }
};
</script>

<style>
.map-header {
  min-height: 30px;
  padding: 10px;
  font-size: 1.1rem;
}
.legend-circle {
  height: 18px;
  width: 18px;
  background-color: rgba(192, 108, 132, 0.5);
  border: 1px solid rgba(192, 108, 132, 1);
  border-radius: 50%;
  display: inline-block;
}

.legend-circle-active {
  height: 25px;
  width: 25px;
  background-color: rgba(37, 206, 247, 0.5);
  border: 1px solid rgba(37, 206, 247, 1);
}
.legend-circle-failed {
  height: 18px;
  width: 18px;
  background-color: rgba(207, 207, 207, 0.5);
  border: 1px solid rgba(207, 207, 207, 1);
}

.mapboxgl-popup-content div {
  padding: 5px !important;
}

.mapboxgl-popup-content {
  border-radius: 25px !important;
}

.my-tooltip {
  text-align: center;
  font: 1rem sans-serif;
  background: #ffffff;
  border-radius: 8px;
  pointer-events: none;
  border: 1px solid #cdcdcd;
  opacity: 1;
  display: block;
  max-width: 300px;
}
.tooltip-title {
  margin-bottom: 5px;
  border-bottom: 1px solid #cdcdcd;
  text-align: center;
  font-weight: bold;
  padding-bottom: 5px;
}
</style>