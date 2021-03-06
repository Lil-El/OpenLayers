<template>
  <div class="wrap">
    <div id="map"></div>
    <tool>
      <form>
        <div id="status">
          Show impacts between <span class="min-year"></span> and
          <span class="max-year"></span>
        </div>

        <label>Minimum year:</label>
        <input
          id="min-year"
          type="range"
          min="1850"
          max="2015"
          step="1"
          value="1850"
        />
        <label>Maximum year:</label>
        <input
          id="max-year"
          type="range"
          min="1850"
          max="2015"
          step="1"
          value="2015"
        />
      </form>
    </tool>
  </div>
</template>

<script>
import Feature from "ol/Feature";
import Map from "ol/Map";
import Point from "ol/geom/Point";
import Stamen from "ol/source/Stamen";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import WebGLPointsLayer from "ol/layer/WebGLPoints";
import { Vector } from "ol/source";
import { fromLonLat } from "ol/proj";
export default {
  //本示例：使用text styole的WebGLPoints动态过滤大量的点几何
  mounted() {
    var vectorSource = new Vector();

    var oldColor = "rgba(242,56,22,0.61)";
    var newColor = "#ffe52c";
    var period = 12; // animation period in seconds
    var animRatio = [
      "^",
      [
        "/",
        [
          "%",
          [
            "+",
            ["time"],
            ["interpolate", ["linear"], ["get", "year"], 1850, 0, 2015, period],
          ],
          period,
        ],
        period,
      ],
      0.5,
    ];

    var style = {
      variables: {
        minYear: 1850,
        maxYear: 2015,
      },
      filter: [
        "between",
        ["get", "year"],
        ["var", "minYear"],
        ["var", "maxYear"],
      ],
      symbol: {
        symbolType: "circle",
        size: [
          "*",
          ["interpolate", ["linear"], ["get", "mass"], 0, 8, 200000, 26],
          ["-", 1.75, ["*", animRatio, 0.75]],
        ],
        color: ["interpolate", ["linear"], animRatio, 0, newColor, 1, oldColor],
        opacity: ["-", 1.0, ["*", animRatio, 0.75]],
      },
    };

    var minYearInput = document.getElementById("min-year");
    var maxYearInput = document.getElementById("max-year");

    function updateMinYear() {
      style.variables.minYear = parseInt(minYearInput.value);
      updateStatusText();
    }
    function updateMaxYear() {
      style.variables.maxYear = parseInt(maxYearInput.value);
      updateStatusText();
    }
    function updateStatusText() {
      var div = document.getElementById("status");
      div.querySelector("span.min-year").textContent = minYearInput.value;
      div.querySelector("span.max-year").textContent = maxYearInput.value;
    }

    minYearInput.addEventListener("input", updateMinYear);
    minYearInput.addEventListener("change", updateMinYear);

    maxYearInput.addEventListener("input", updateMaxYear);
    maxYearInput.addEventListener("change", updateMaxYear);
    updateStatusText();

    // 为source添加feature（mass，year，geometry）
    var client = new XMLHttpRequest();
    client.open(
      "GET",
      "https://openlayers.org/en/latest/examples/data/csv/meteorite_landings.csv"
    );
    client.onload = function () {
      var csv = client.responseText;
      var features = [];

      var prevIndex = csv.indexOf("\n") + 1; // scan past the header line

      var curIndex;
      while ((curIndex = csv.indexOf("\n", prevIndex)) != -1) {
        var line = csv.substr(prevIndex, curIndex - prevIndex).split(",");
        prevIndex = curIndex + 1;

        var coords = fromLonLat([parseFloat(line[4]), parseFloat(line[3])]);
        if (isNaN(coords[0]) || isNaN(coords[1])) {
          // guard against bad data
          continue;
        }

        features.push(
          new Feature({
            mass: parseFloat(line[1]) || 0,
            year: parseInt(line[2]) || 0,
            geometry: new Point(coords),
          })
        );
      }

      vectorSource.addFeatures(features);
    };
    client.send();

    var map = new Map({
      layers: [
        new TileLayer({
          source: new Stamen({
            layer: "toner",
          }),
        }),
        new WebGLPointsLayer({
          style: style,
          source: vectorSource,
          disableHitDetection: true,
        }),
      ],
      target: "map",
      view: new View({
        center: [0, 0],
        zoom: 2,
      }),
    });

    function animate() {
      map.render();
      window.requestAnimationFrame(animate);
    }
    animate();
  },
};
</script>

<style>
</style>