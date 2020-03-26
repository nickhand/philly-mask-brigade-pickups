<template>
  <div class="map-overlay-container" v-if="item">
    <div class="d-flex overlay-header justify-content-between align-items-center">
      <div
        class="font-weight-bold"
      >{{ item.properties["Street Name & Number (i.e. 1234 Broad St)"] }} {{ item.properties["Apartment / Unit Number"] }} {{ item.properties["City"] }} {{ item.properties["ZIP Code"] }}</div>
      <div
        class="ml-1 neighborhood-marker text-center"
        :class="getRegionClass(item)"
      >{{ item.properties["Delivery Region"] }}</div>
    </div>
    <div class="overlay-body">
      <div class="d-flex overlay-line">
        <div class="overlay-label mr-1">Timestamp:</div>
        <div>{{item.properties['Timestamp']}}</div>
      </div>
      <div class="d-flex overlay-line">
        <div class="overlay-label mr-1">Email:</div>
        <div>{{item.properties['Email Address']}}</div>
      </div>
      <div class="d-flex overlay-line">
        <div class="overlay-label mr-1">Instructions/Contact Info:</div>
        <div>{{item.properties['Additional pickup instructions or other contact info. ']}}</div>
      </div>
      <div class="d-flex overlay-line">
        <div class="overlay-label mr-1">Number of Masks:</div>
        <div>{{item.properties['Approximate number of unused masks']}}</div>
      </div>
      <div class="d-flex overlay-line">
        <div class="overlay-label mr-1">Condition:</div>
        <div>{{item.properties['Condition of masks']}}</div>
      </div>
      <div
        class="d-flex overlay-line"
        v-if="item.properties['Would you like your donation to go to a particular area hospital or healthcare provider?']"
      >
        <div class="overlay-label mr-1">Desired Hospital:</div>
        <div>{{item.properties['Would you like your donation to go to a particular area hospital or healthcare provider?']}}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MapOverlay",
  props: ["item"],
  methods: {
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
.overlay-header {
  font-size: 1.2rem;
}
.overlay-body {
  font-weight: normal;
  font-size: 1.1rem;
}
.overlay-label {
  font-weight: 700;
  min-width: 250px;
}
</style>