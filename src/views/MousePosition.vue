<template>
  <div class="wrap">
    <div id="map"></div>
    <tool>
      <div id="mouse-position"></div>
      <form>
        <label>Projection </label>
        <select id="projection">
          <option value="EPSG:4326">EPSG:4326</option>
          <option value="EPSG:3857">EPSG:3857</option>
        </select>
        <label>Precision </label>
        <input id="precision" type="number" min="0" max="12" value="4" />
      </form>
    </tool>
  </div>
</template>

<script>
import Map from "ol/Map";
import MousePosition from "ol/control/MousePosition";
import OSM from "ol/source/OSM";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import { createStringXY } from "ol/coordinate";
import { defaults as defaultControls } from "ol/control";
export default {
  mounted() {
    var mousePositionControl = new MousePosition({
      coordinateFormat: createStringXY(4),
      projection: "EPSG:4326",
      // comment the following two lines to have the mouse position
      // be placed within the map.
      className: "custom-mouse-position",
      target: document.getElementById("mouse-position"),
      undefinedHTML: "&nbsp;",
    });

    var map = new Map({
      controls: defaultControls().extend([mousePositionControl]),
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
      ],
      target: "map",
      view: new View({
        center: [0, 0],
        zoom: 2,
      }),
    });

    this.$nextTick(() => {
      var projectionSelect = document.getElementById("projection");
      projectionSelect.addEventListener("change", function (event) {
        mousePositionControl.setProjection(event.target.value);
      });

      var precisionInput = document.getElementById("precision");
      precisionInput.addEventListener("change", function (event) {
        var format = createStringXY(event.target.valueAsNumber);
        mousePositionControl.setCoordinateFormat(format);
      });
    });
  },
};
</script>

<style>
</style>