<template>
  <div class="wrap">
    <div id="map"></div>
    <tool>
      <form class="form-inline">
        <label for="epsg-query">Search projection:</label>
        <input
          type="text"
          id="epsg-query"
          placeholder="4326, 27700, 3031, US National Atlas, Swiss, France, ..."
          class="form-control"
          size="50"
        />
        <button id="epsg-search" class="btn">Search</button>
        <span id="epsg-result"></span>
        <div>
          <label for="render-edges">
            Render reprojection edges
            <input type="checkbox" id="render-edges" />
          </label>
          <label for="show-graticule">
            &nbsp;&nbsp;&nbsp;Show graticule
            <input type="checkbox" id="show-graticule" />
          </label>
        </div>
      </form>
    </tool>
  </div>
</template>

<script>
import Graticule from "ol/layer/Graticule";
import Map from "ol/Map";
import OSM from "ol/source/OSM";
import Stroke from "ol/style/Stroke";
import TileImage from "ol/source/TileImage";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import proj4 from "proj4";
import { applyTransform } from "ol/extent";
import { get as getProjection, getTransform } from "ol/proj";
import { register } from "ol/proj/proj4";
export default {
  mounted() {
    /**
     * 本示例通过在EPSG.io数据库中进行搜索，显示了从OSM（EPSG：3857）到任意投影的客户端栅格重新投影功能。
     */
    var graticule = new Graticule({
      // the style to use for the lines, optional.
      strokeStyle: new Stroke({
        color: "rgba(255,120,0,0.9)",
        width: 2,
        lineDash: [0.5, 4],
      }),
      showLabels: true,
      visible: false,
      wrapX: false,
    });

    var map = new Map({
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
        graticule,
      ],
      target: "map",
      view: new View({
        projection: "EPSG:3857",
        center: [0, 0],
        zoom: 1,
      }),
    });

    var queryInput = document.getElementById("epsg-query");
    var searchButton = document.getElementById("epsg-search");
    var resultSpan = document.getElementById("epsg-result");
    var renderEdgesCheckbox = document.getElementById("render-edges");
    var showGraticuleCheckbox = document.getElementById("show-graticule");

    function setProjection(code, name, proj4def, bbox) {
      if (
        code === null ||
        name === null ||
        proj4def === null ||
        bbox === null
      ) {
        resultSpan.innerHTML = "Nothing usable found, using EPSG:3857...";
        map.setView(
          new View({
            projection: "EPSG:3857",
            center: [0, 0],
            zoom: 1,
          })
        );
        return;
      }

      resultSpan.innerHTML = "(" + code + ") " + name;

      var newProjCode = "EPSG:" + code;
      proj4.defs(newProjCode, proj4def);
      register(proj4);
      var newProj = getProjection(newProjCode);
      var fromLonLat = getTransform("EPSG:4326", newProj);

      var worldExtent = [bbox[1], bbox[2], bbox[3], bbox[0]];
      newProj.setWorldExtent(worldExtent);

      // approximate calculation of projection extent,
      // checking if the world extent crosses the dateline
      if (bbox[1] > bbox[3]) {
        worldExtent = [bbox[1], bbox[2], bbox[3] + 360, bbox[0]];
      }
      var extent = applyTransform(worldExtent, fromLonLat, undefined, 8);
      newProj.setExtent(extent);
      var newView = new View({
        projection: newProj,
      });
      map.setView(newView);
      newView.fit(extent);
    }

    function search(query) {
      resultSpan.innerHTML = "Searching ...";
      fetch("https://epsg.io/?format=json&q=" + query)
        .then(function (response) {
          return response.json();
        })
        .then(function (json) {
          var results = json["results"];
          if (results && results.length > 0) {
            for (var i = 0, ii = results.length; i < ii; i++) {
              var result = results[i];
              if (result) {
                var code = result["code"];
                var name = result["name"];
                var proj4def = result["proj4"];
                var bbox = result["bbox"];
                if (
                  code &&
                  code.length > 0 &&
                  proj4def &&
                  proj4def.length > 0 &&
                  bbox &&
                  bbox.length == 4
                ) {
                  setProjection(code, name, proj4def, bbox);
                  return;
                }
              }
            }
          }
          setProjection(null, null, null, null);
        });
    }

    searchButton.onclick = function (event) {
      search(queryInput.value);
      event.preventDefault();
    };

    renderEdgesCheckbox.onchange = function () {
      map.getLayers().forEach(function (layer) {
        if (layer instanceof TileLayer) {
          var source = layer.getSource();
          if (source instanceof TileImage) {
            source.setRenderReprojectionEdges(renderEdgesCheckbox.checked);
          }
        }
      });
    };

    showGraticuleCheckbox.onchange = function () {
      graticule.setVisible(showGraticuleCheckbox.checked);
    };
  },
};
</script>

<style>
</style>