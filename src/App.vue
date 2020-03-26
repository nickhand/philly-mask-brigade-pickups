<template>
  <div id="app" class="d-flex flex-row">
    <!-- Map Overlay -->
    <div v-if="showFullScreenMapOverlay" class="map-overlay">
      <div class="h-100">
        <div class="d-flex justify-content-end">
          <a @click="hideFullScreenMapOverlay" style="cursor: pointer">
            <i class="fas fa-times"></i>
          </a>
        </div>
        <MapOverlay :item="clickedItem" />
      </div>
    </div>
    <!-- Sidebar -->
    <div class="sidebar sidebar-pad flex-grow-1">
      <div v-for="(pickups, i) in pickupsData" :key="i">
        <div
          class="heading d-flex justify-content-between align-items-center"
          @click="togglePanel(i)"
        >
          <h1>{{ pickups.header }} Pickups</h1>
          <a v-show="pickups.show">
            <i class="fas fa-caret-up fa-2x collapse-icon"></i>
          </a>
          <a v-show="!pickups.show">
            <i class="fas fa-caret-down fa-2x collapse-icon"></i>
          </a>
        </div>
        <div id="listings" class="listings" v-show="pickups.show">
          <div
            class="mb-3 listing-entry"
            v-for="item in pickups.data"
            :key="item['Timestamp']"
            @click="handleClickedListing(item)"
          >
            <div class="d-flex justify-content-between align-items-center">
              <div
                class="listing-title"
              >{{ item.properties["Street Name & Number (i.e. 1234 Broad St)"] }} {{ item.properties["City"] }} {{ item.properties["ZIP Code"] }}</div>
              <div
                class="ml-1 neighborhood-marker text-center"
                :class="getRegionClass(item)"
              >{{ item.properties["Delivery Region"] }}</div>
            </div>
            <div>{{ item.properties["Approximate number of unused masks"] }} masks</div>
          </div>
        </div>
      </div>
    </div>

    <div class="right-half sidebar-pad">
      <!-- Map -->
      <Map class="map" :pickupFeatures="pickupFeatures" ref="pickupsMap" />

      <!-- Map Overlay -->
      <div v-if="showMapOverlay" class="map-overlay">
        <div class="h-100">
          <div class="d-flex justify-content-end">
            <a @click="hideMapOverlay" style="cursor: pointer">
              <i class="fas fa-times"></i>
            </a>
          </div>
          <MapOverlay :item="clickedItem" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Map from "./components/Map.vue";
import MapOverlay from "./components/MapOverlay.vue";
import axios from "axios";

export default {
  name: "App",
  components: {
    Map,
    MapOverlay
  },
  data() {
    return {
      pickupFeatures: [],
      pickupsData: [
        { header: "Active", data: [], show: true },
        { header: "Failed", data: [], show: false },
        { header: "Completed", data: [], show: false }
      ],
      showMapOverlay: false,
      showFullScreenMapOverlay: false,
      clickedItem: null
    };
  },
  created() {
    // Load the pickups
    axios.get(`https://philly-mask-brigade.s3.amazonaws.com/pickups.json`).then(
      response => {
        this.pickupFeatures = response.data.features;
        this.pickupsData[0].data = this.activePickups;
        this.pickupsData[1].data = this.failedPickups;
        this.pickupsData[2].data = this.completedPickups;
      },
      error => {
        console.log("ERROR", error);
      }
    );
  },
  computed: {
    activePickups() {
      return this.pickupFeatures.filter(
        item =>
          (item.properties["Success?"] != "Failed") &
          (item.properties["Success?"] != "Done")
      );
    },
    failedPickups() {
      return this.pickupFeatures.filter(
        item => item.properties["Success?"] === "Failed"
      );
    },
    completedPickups() {
      return this.pickupFeatures.filter(
        item => item.properties["Success?"] === "Done"
      );
    }
  },
  methods: {
    getWidth() {
      return window.screen.height;
    },
    hideMapOverlay() {
      if (this.getWidth() > 768) this.showMapOverlay = false;
      else {
        this.showFullScreenMapOverlay = false;
        $("body").addClass("overflow-scroll");
        $("body").removeClass("no-overflow");
        consol;
      }
    },
    handleClickedListing(currentFeature) {
      // Fly to listing on map
      let childMap = this.$refs.pickupsMap;
      let mapboxMap = childMap.$refs.pickupsMap.map;
      let coords = currentFeature.geometry.coordinates;
      coords[1] -= 0.0035;
      mapboxMap.flyTo({
        center: coords,
        zoom: 15
      });

      // show popup
      if (this.getWidth() > 768) this.showMapOverlay = true;
      else {
        this.showFullScreenMapOverlay = true;
        $("body").removeClass("overflow-scroll");
        $("body").addClass("no-overflow");
      }
      this.clickedItem = currentFeature;
    },
    togglePanel(i) {
      this.pickupsData[i].show = !this.pickupsData[i].show;
    },
    getRegionClass(item) {
      let hood = item.properties["Delivery Region"];
      if (hood == "Far Northeast") return "far-northeast";
      else if (hood == "Germantown/Chestnut Hill") return "chestnut-hill";
      else if (hood == "Lower Northeast") return "lower-northeast";
      else if (hood == "Olney/Oak Lane") return "olney";
      else if (hood == "Roxborough/Manayunk") return "manayunk";
      else if (hood == "Upper North Philadelphia") return "upper-north-philly";
      else if (hood == "Kensington/River Wards") return "river-wards";
      else if (hood == "West Philadelphia") return "west-philly";
      else if (hood == "Lower North Philadelphia") return "lower-north-philly";
      else if (hood == "Center City") return "center-city";
      else if (hood == "Southwest Philadelphia") return "southwest-philly";
      else if (hood == "South Philadelphia") return "south-philly";
      else return "outside-philly";
    }
  }
};
</script>

<style>
.heading:hover {
  cursor: pointer;
}
.map-overlay {
  z-index: 1000;
  color: black;
  background: #fff;
  position: absolute;
  top: 62.5%;
  bottom: 0;
  height: 300px;
  width: 100%;
  /* border-radius: 25px 25px 0px 0px; */
  padding: 10px;
  border: 4px solid #cfcfcf;
  font-size: 1.5rem;
  overflow-y: scroll;
}
.collapse-icon {
  cursor: pointer;
  vertical-align: middle;
}
.far-northeast {
  background: rgba(18, 237, 237, 0.5);
  border: 2px solid rgba(18, 237, 237, 0.8);
}

.chestnut-hill {
  background: rgba(18, 237, 164, 0.5);
  border: 2px solid rgba(18, 237, 164, 0.8);
}

.lower-northeast {
  background: rgba(18, 237, 18, 0.5);
  border: 2px solid rgba(18, 237, 18, 0.8);
}

.olney {
  background: rgba(165, 237, 18, 0.5);
  border: 2px solid rgba(165, 237, 18, 0.8);
}

.manayunk {
  background: rgba(237, 237, 18, 0.5);
  border: 2px solid rgba(237, 237, 18, 0.8);
}

.upper-north-philly {
  background: rgba(237, 189, 18, 0.5);
  border: 2px solid rgba(237, 189, 18, 0.8);
}

.river-wards {
  background: rgba(237, 125, 18, 0.5);
  border: 2px solid rgba(237, 125, 18, 0.8);
}
.west-philly {
  background: rgba(237, 18, 18, 0.5);
  border: 2px solid rgba(237, 18, 18, 0.8);
}
.lower-north-philly {
  background: rgba(237, 18, 148, 0.5);
  border: 2px solid rgba(237, 18, 148, 0.8);
}
.center-city {
  background: rgba(153, 18, 237, 0.5);
  border: 2px solid rgba(153, 18, 237, 0.8);
}
.southwest-philly {
  background: rgba(18, 18, 237, 0.5);
  border: 2px solid rgba(18, 18, 237, 0.8);
}
.south-philly {
  background: rgba(18, 157, 237, 0.5);
  border: 2px solid rgba(18, 157, 237, 0.8);
}

.outside-philly {
  background: #666;
  color: white;
  border: 2px solid #444;
}

.listing-entry {
  border-bottom: 1px solid #cfcfcf;
}
.neighborhood-marker {
  opacity: 0.8;
  padding: 3px;
  border-radius: 5px;
  margin: 3px;
}

.north-philadelphia {
  background: #2176d2;
  color: white;
  border: 2px solid blue;
}

#app {
  color: #404040;
  font: 400 15px/22px "Source Sans Pro", "Helvetica Neue", Sans-serif;
  margin: 0;
  padding: 0;
  -webkit-font-smoothing: antialiased;
  flex-shrink: 0;
  flex-grow: 1;
  height: 100%;
  min-height: 800px;
}

* {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

.sidebar {
  border-right: 1px solid rgba(0, 0, 0, 0.25);
  overflow-y: scroll;
  max-height: 800px;
  flex: 1 1 0;
}
.sidebar-pad {
  padding: 0px 5px 0px 5px;
}

.right-half {
  overflow: hidden;
  flex: 1.25 1 0;
  position: relative;
}

.map {
  height: 100%;
}

h1 {
  font-size: 22px;
  margin: 0;
  font-weight: 400;
  line-height: 20px;
  padding: 20px 2px;
}

a {
  color: #404040;
  text-decoration: none;
}

a:hover {
  color: #101010;
}

.heading {
  background: #fff;
  border-bottom: 1px solid #eee;
  min-height: 60px;
  line-height: 60px;
  padding: 0 10px;
  background-color: rgb(52, 51, 50);
  color: #fff;
  font-size: 0.9rem;
}

.listings {
  height: 100%;
  overflow: auto;
  padding: 0 10px;
  padding-bottom: 30px;
}

.listing-title {
  font-weight: 700;
  font-size: 1.1rem;
}

.listings .listing-entry:hover .listing-title,
.listings .listing-entry .listing-title:hover {
  color: #666;
  cursor: pointer;
}
.listings .listing-entry:hover {
  background-color: #f0f0f0;
  cursor: pointer;
}
::-webkit-scrollbar {
  width: 5px;
  height: 5px;
  border-left: 0;
  background: #cfcfcf;
}
::-webkit-scrollbar-track {
  background: none;
}
::-webkit-scrollbar-thumb {
  background: #444;
  border-radius: 0;
}

.clearfix {
  display: block;
}
.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}

/* Marker tweaks */
.mapboxgl-popup {
  padding-bottom: 50px;
}

.mapboxgl-popup-close-button {
  display: none;
}
.mapboxgl-popup-content {
  font: 400 15px/22px "Source Sans Pro", "Helvetica Neue", Sans-serif;
  padding: 0;
  width: 180px;
}
.mapboxgl-popup-content-wrapper {
  padding: 1%;
}
.mapboxgl-popup-content h3 {
  background: #91c949;
  color: #fff;
  margin: 0;
  display: block;
  padding: 10px;
  border-radius: 3px 3px 0 0;
  font-weight: 700;
  margin-top: -15px;
}

.mapboxgl-popup-content h4 {
  margin: 0;
  display: block;
  padding: 10px 10px 10px 10px;
  font-weight: 400;
}

.mapboxgl-popup-content div {
  padding: 10px;
}

.mapboxgl-container .leaflet-marker-icon {
  cursor: pointer;
}

.mapboxgl-popup-anchor-top > .mapboxgl-popup-content {
  margin-top: 15px;
}

.mapboxgl-popup-anchor-top > .mapboxgl-popup-tip {
  border-bottom-color: #91c949;
}

.no-overflow {
  overflow: hidden;
}
.overflow-scroll {
  overflow: scroll;
}
@media (max-width: 768px) {
  .map {
    height: 600px;
  }
}

@media (max-width: 1024px) {
  .right-half {
    width: 100% !important;
    height: 100% !important;
    flex: 1 !important;
    margin-top: 1rem;
    border-top: 5px solid #cfcfcf;
  }
  .map {
    height: 1000px;
  }
  #app {
    flex-direction: column !important;
  }
  .map-overlay {
    top: 0;
    height: 100%;
    position: absolute;
  }
  .sidebar {
    flex: 1 !important;
  }
}
</style>
