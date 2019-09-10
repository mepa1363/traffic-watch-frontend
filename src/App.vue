<template>
  <v-app>
    <v-content>
      <div id="map"></div>
      <v-card
        v-if="showSummaryInfo"
        :loading="isSummaryLoading"
        max-width="344"
        class="summary-info mx-left ml-2 mt-2"
      >
        <v-card-title class="pb-0">Summary</v-card-title>
        <div class="caption px-4">calculated from data across the city</div>
        <v-card-text>
          <v-row>
            <v-list dense style="width: 100%;" class="mx-7">
              <v-list-item-group v-model="summaryMapLayer" mandatory color="indigo">
                <v-list-item v-for="(info, index) in summaryInfo" :key="index" :value="info.label">
                  <v-list-item-icon>
                    <v-icon v-if="info.label === 'Vehicle'">mdi-car</v-icon>
                    <v-icon v-if="info.label === 'Pedestrian'">mdi-account</v-icon>
                    <v-icon v-if="info.label === 'Bike'">mdi-bike</v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>{{ info.label }}: {{ info.sum.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") }}</v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
            <v-list v-if="!isMapLoading" class="mx-auto">
              <v-list-item>
                <span class="app-map-legend-title">Low Volume</span>
                <v-list-item-content>
                  <v-list-item-title>
                    <span class="app-map-legend-item" style="background: #feebe2"></span>
                    <span class="app-map-legend-item" style="background: #fcc5c0"></span>
                    <span class="app-map-legend-item" style="background: #fa9fb5"></span>
                    <span class="app-map-legend-item" style="background: #f768a1"></span>
                    <span class="app-map-legend-item" style="background: #c51b8a"></span>
                    <span class="app-map-legend-item" style="background: #7a0177"></span>
                  </v-list-item-title>
                </v-list-item-content>
                <span class="app-map-legend-title">High Volume</span>
              </v-list-item>
            </v-list>
            <v-btn-toggle class="mx-auto my-5" v-model="temporalWindow" mandatory>
              <v-btn small temporalWindow value="today">Today</v-btn>
              <v-btn small temporalWindow value="past-week">Past Week</v-btn>
              <v-btn small temporalWindow value="past-month">Past Month</v-btn>
            </v-btn-toggle>
          </v-row>
        </v-card-text>
      </v-card>
      <v-card
        v-if="showCameraInfo"
        :loading="isCameraInfoLoading"
        max-width="344"
        class="camera-info mx-left ml-2 mt-2"
      >
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title>{{ cameraInfo.address }}</v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <v-btn small icon @click="showCameraInfo = false, showSummaryInfo = true">
              <v-icon small color="grey lighten-1">mdi-close</v-icon>
            </v-btn>
          </v-list-item-action>
        </v-list-item>
        <v-row align="center" justify="center">
          <v-img
            v-if="imageTags.some(item => item.tag === 'Original' && !item.disabled) && selectedImage==='Original'"
            :src="this.cameraInfo.image"
            class="grey lighten-2"
            max-width="344"
            max-height="240"
          ></v-img>
          <v-img
            v-if="imageTags.some(item => item.tag === 'Tagged' && !item.disabled) && selectedImage==='Tagged'"
            :src="this.cameraInfo.processedImage"
            class="grey lighten-2"
            max-width="344"
            max-height="240"
          ></v-img>
          <v-img
            v-if="imageTags.some(item => item.tag === 'Vehicles Trail' && !item.disabled) && selectedImage==='Vehicles Trail'"
            :src="this.cameraInfo.vehiclesTrailImage"
            class="grey lighten-2"
            max-width="344"
            max-height="240"
          ></v-img>
          <v-img
            v-if="imageTags.some(item => item.tag === 'Pedestrians Trail' && !item.disabled) && selectedImage==='Pedestrians Trail'"
            :src="this.cameraInfo.pedestriansTrailImage"
            class="grey lighten-2"
            max-width="344"
            max-height="240"
          ></v-img>
        </v-row>
        <v-row justify="space-around">
          <v-col cols="12" class="pb-0">
            <v-sheet class="mx-auto" elevation="0" max-width="344">
              <v-chip-group
                v-model="selectedImage"
                show-arrows
                mandatory
                active-class="primary--text"
              >
                <v-chip
                  v-for="item in imageTags"
                  :disabled="item.disabled"
                  :key="item.tag"
                  :value="item.tag"
                >{{ item.tag }}</v-chip>
              </v-chip-group>
            </v-sheet>
          </v-col>
        </v-row>
        <v-card-text>
          <v-row class="mx-auto">
            <v-col
              class="pa-0"
              cols="4"
              v-for="object in cameraInfo.objects"
              :key="`${object.label}`"
            >
              <v-col class="py-0" cols="12">{{ object.label }}</v-col>
              <v-col
                class="py-0 headline"
                cols="12"
              >{{ object.sum.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") }}</v-col>
            </v-col>
          </v-row>
          <v-row class="mx-auto mt-5" v-if="selectedChart === 'Observations'">
            <HourlyTrendOverToday
              v-if="temporalWindow === 'today'"
              :key="cameraInfo.id"
              :cameraId="cameraInfo.id"
            />
            <DailyTrendOverPastWeek
              v-if="temporalWindow === 'past-week'"
              :key="cameraInfo.id"
              :cameraId="cameraInfo.id"
            />
            <DailyTrendOverPastMonth
              v-if="temporalWindow === 'past-month'"
              :key="cameraInfo.id"
              :cameraId="cameraInfo.id"
            />
            <v-btn-toggle class="mx-auto" v-model="temporalWindow" mandatory>
              <v-btn small temporalWindow value="today">Today</v-btn>
              <v-btn small temporalWindow value="past-week">Past Week</v-btn>
              <v-btn small temporalWindow value="past-month">Past Month</v-btn>
            </v-btn-toggle>
          </v-row>
          <v-row class="mx-auto mt-5" v-if="selectedChart === 'Predictions'">
            <MotorVehicleTrafficTrend :cameraId="cameraInfo.id" :key="cameraInfo.id" />
          </v-row>
          <v-row class="mx-auto" justify="space-around">
            <v-col cols="12" class="pb-0">
              <v-sheet class="mx-auto" elevation="0" max-width="344">
                <v-chip-group
                  v-model="selectedChart"
                  show-arrows
                  mandatory
                  active-class="primary--text"
                >
                  <v-chip v-for="item in chartTags" :key="item.tag" :value="item.tag">{{ item.tag }}</v-chip>
                </v-chip-group>
              </v-sheet>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
      <v-overlay :value="isMapLoading">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
      <span class="app-logo">
        <img src="./assets/logo.svg" />
      </span>
    </v-content>
  </v-app>
</template>


<script>
import MotorVehicleTrafficTrend from "./components/MotorVehicleTrafficTrend";
import HourlyTrendOverToday from "./components/HourlyTrendOverToday";
import DailyTrendOverPastWeek from "./components/DailyTrendOverPastWeek";
import DailyTrendOverPastMonth from "./components/DailyTrendOverPastMonth";

import mapboxgl from "mapbox-gl";
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
import axios from "axios";

export default {
  name: "App",
  components: {
    MotorVehicleTrafficTrend,
    HourlyTrendOverToday,
    DailyTrendOverPastWeek,
    DailyTrendOverPastMonth
  },
  data: () => ({
    showSummaryInfo: true,
    isSummaryLoading: false,
    summaryInfo: {},
    cameraInfo: {},
    showCameraInfo: false,
    isCameraInfoLoading: false,
    temporalWindow: "today",
    map: null,
    summaryMapLayer: "Vehicle",
    isMapLoading: false,
    selectedImage: null,
    selectedChart: null,
    chartTags: [{ tag: "Observations" }, { tag: "Predictions" }],
    predictionTemporalWindow: 48
  }),
  methods: {
    loadSummaryInfo(temporalWindow) {
      this.isSummaryLoading = true;
      axios
        .get(`${process.env.VUE_APP_API}/camera/all/summary/${temporalWindow}`)
        .then(response => {
          this.summaryInfo = response.data;
          this.isSummaryLoading = false;
        });
    },
    loadSummaryMap(temporalWindow) {
      this.isMapLoading = true;
      axios
        .get(`${process.env.VUE_APP_API}/community-summary/${temporalWindow}`)
        .then(response => {
          const vehicles = response.data.features.filter(
            item => item.properties.label === "Vehicle"
          );
          const pedestrians = response.data.features.filter(
            item => item.properties.label === "Pedestrian"
          );
          const bikes = response.data.features.filter(
            item => item.properties.label === "Bike"
          );
          const vehiclesMinMaxRate = this.findMinMaxRate(vehicles);
          const vehiclesInterval =
            (vehiclesMinMaxRate.max - vehiclesMinMaxRate.min) / 5;

          const bikesMinMaxRate = this.findMinMaxRate(bikes);
          const bikesInterval = (bikesMinMaxRate.max - bikesMinMaxRate.min) / 5;

          const pedestriansMinMaxRate = this.findMinMaxRate(pedestrians);
          const pedestriansInterval =
            (pedestriansMinMaxRate.max - pedestriansMinMaxRate.min) / 5;

          this.addLayer(
            `vehicle-${temporalWindow}`,
            vehicles,
            vehiclesMinMaxRate.min,
            vehiclesInterval,
            this.summaryMapLayer === "Vehicle" ? "visible" : "none"
          );
          this.addLayer(
            `pedestrian-${temporalWindow}`,
            pedestrians,
            pedestriansMinMaxRate.min,
            pedestriansInterval,
            this.summaryMapLayer === "Pedestrian" ? "visible" : "none"
          );
          this.addLayer(
            `bike-${temporalWindow}`,
            bikes,
            bikesMinMaxRate.min,
            bikesInterval,
            this.summaryMapLayer === "Bike" ? "visible" : "none"
          );
          this.isMapLoading = false;
        });
    },
    findMinMaxRate(array) {
      return {
        max: Math.max(...array.map(item => item.properties.rate), 0),
        min: Math.min(...array.map(item => item.properties.rate), 0)
      };
    },
    addLayer(layerId, features, min, interval, visibility) {
      this.map.addLayer(
        {
          id: `${layerId}`,
          type: "fill",
          source: {
            type: "geojson",
            data: {
              type: "FeatureCollection",
              features: features
            }
          },
          paint: {
            "fill-color": [
              "interpolate",
              ["linear"],
              ["get", "rate"],
              min,
              "#feebe2",
              min + interval,
              "#fcc5c0",
              min + 2 * interval,
              "#fa9fb5",
              min + 3 * interval,
              "#f768a1",
              min + 4 * interval,
              "#c51b8a",
              min + 5 * interval,
              "#7a0177"
            ],
            "fill-opacity": 1
          },
          layout: {
            visibility: visibility
          }
        },
        "waterway"
      );
    },
    loadCameraInfo(id, address, image, update) {
      this.isCameraInfoLoading = true;
      axios
        .get(
          `${process.env.VUE_APP_API}/camera/${id}/summary/${this.temporalWindow}`
        )
        .then(response => {
          if (update) {
            this.$set(this.cameraInfo, "objects", response.data);
          } else {
            this.cameraInfo = {
              id: id,
              address: address,
              image: image,
              objects: response.data
            };
          }
          this.showSummaryInfo = false;
          this.isCameraInfoLoading = false;
          this.showCameraInfo = true;
        });
    },
    loadImageAnalysisResult(id, image) {
      axios
        .post(process.env.VUE_APP_ANALYSIS_API, {
          image: image,
          createImage: true,
          summerize: true
        })
        .then(response => {
          this.$set(
            this.cameraInfo,
            "processedImage",
            `data:image/png;base64, ${response.data.image}`
          );
        });
      axios
        .post(process.env.VUE_APP_HISTORICAL_API, {
          cameraId: id,
          label: "car",
          color: "(255,0,0)"
        })
        .then(response => {
          this.$set(
            this.cameraInfo,
            "vehiclesTrailImage",
            `data:image/jpg;base64, ${response.data}`
          );
        });
      axios
        .post(process.env.VUE_APP_HISTORICAL_API, {
          cameraId: id,
          label: "person",
          color: "(0,255,0)"
        })
        .then(response => {
          this.$set(
            this.cameraInfo,
            "pedestriansTrailImage",
            `data:image/jpg;base64, ${response.data}`
          );
        });
    }
  },
  watch: {
    temporalWindow(newValue, oldValue) {
      if (this.map.getLayer(`vehicle-${oldValue}`)) {
        this.map.removeLayer(`vehicle-${oldValue}`);
        this.map.removeSource(`vehicle-${oldValue}`);
      }
      if (this.map.getLayer(`pedestrian-${oldValue}`)) {
        this.map.removeLayer(`pedestrian-${oldValue}`);
        this.map.removeSource(`pedestrian-${oldValue}`);
      }
      if (this.map.getLayer(`bike-${oldValue}`)) {
        this.map.removeLayer(`bike-${oldValue}`);
        this.map.removeSource(`bike-${oldValue}`);
      }
      if (this.showSummaryInfo) {
        this.loadSummaryInfo(newValue);
      }
      if (this.showCameraInfo) {
        this.loadCameraInfo(
          this.cameraInfo.id,
          this.cameraInfo.address,
          this.cameraInfo.image,
          true
        );
      }
      this.loadSummaryMap(newValue);
    },
    summaryMapLayer(value) {
      if (value === "Vehicle") {
        this.map.setLayoutProperty(
          `vehicle-${this.temporalWindow}`,
          "visibility",
          "visible"
        );
        this.map.setLayoutProperty(
          `pedestrian-${this.temporalWindow}`,
          "visibility",
          "none"
        );
        this.map.setLayoutProperty(
          `bike-${this.temporalWindow}`,
          "visibility",
          "none"
        );
      } else if (value === "Pedestrian") {
        this.map.setLayoutProperty(
          `vehicle-${this.temporalWindow}`,
          "visibility",
          "none"
        );
        this.map.setLayoutProperty(
          `pedestrian-${this.temporalWindow}`,
          "visibility",
          "visible"
        );
        this.map.setLayoutProperty(
          `bike-${this.temporalWindow}`,
          "visibility",
          "none"
        );
      } else if (value === "Bike") {
        this.map.setLayoutProperty(
          `pedestrian-${this.temporalWindow}`,
          "visibility",
          "none"
        );
        this.map.setLayoutProperty(
          `vehicle-${this.temporalWindow}`,
          "visibility",
          "none"
        );
        this.map.setLayoutProperty(
          `bike-${this.temporalWindow}`,
          "visibility",
          "visible"
        );
      }
    }
  },
  computed: {
    imageTags() {
      return [
        {
          tag: "Original",
          disabled: !this.cameraInfo.image
        },
        {
          tag: "Tagged",
          disabled: !this.cameraInfo.processedImage
        },
        {
          tag: "Vehicles Trail",
          disabled: !this.cameraInfo.vehiclesTrailImage
        },
        {
          tag: "Pedestrians Trail",
          disabled: !this.cameraInfo.pedestriansTrailImage
        }
      ];
    }
  },
  mounted() {
    mapboxgl.accessToken =
      "pk.eyJ1IjoibWVwYTEzNjMiLCJhIjoiY2p6cnk4anB1MHp4bDNicWp3bjA0djJ2eSJ9.Fsu_4-30MTcyyChKfgjR1Q";
    const map = new mapboxgl.Map({
      container: "map",
      center: [-114.0424, 51.017],
      zoom: 10.5,
      bearing: 0,
      pitch: 0,
      style: "mapbox://styles/mapbox/dark-v10",
      antialias: true
    });
    map.addControl(
      new MapboxGeocoder({
        accessToken: mapboxgl.accessToken,
        mapboxgl: mapboxgl
      })
    );
    map.addControl(new mapboxgl.NavigationControl(), "bottom-right");

    map.on("load", () => {
      const layers = map.getStyle().layers;
      let firstSymbolId;
      for (let layer of layers) {
        if (layer.type === "symbol") {
          firstSymbolId = layer.id;
          break;
        }
      }
      map.addLayer(
        {
          id: "3d-buildings",
          source: "composite",
          "source-layer": "building",
          filter: ["==", "extrude", "true"],
          type: "fill-extrusion",
          minzoom: 15,
          paint: {
            "fill-extrusion-color": "#aaa",
            "fill-extrusion-height": [
              "interpolate",
              ["linear"],
              ["zoom"],
              15,
              0,
              15.05,
              ["get", "height"]
            ],
            "fill-extrusion-base": [
              "interpolate",
              ["linear"],
              ["zoom"],
              15,
              0,
              15.05,
              ["get", "min_height"]
            ],
            "fill-extrusion-opacity": 0.6
          }
        },
        firstSymbolId
      );
      axios.get(`${process.env.VUE_APP_API}/camera`).then(response => {
        map.addLayer({
          id: "cameras",
          type: "symbol",
          source: {
            type: "geojson",
            data: {
              type: "FeatureCollection",
              features: response.data.features
            }
          },
          layout: {
            "icon-image": "attraction-15",
            "icon-size": 1
          }
        });

        map.on("click", "cameras", e => {
          const id = e.features[0].id;
          const address = e.features[0].properties.address;
          const image = `http://trafficcam.calgary.ca/loc${id}.jpg`;
          this.selectedImage = "Original";
          this.selectedChart = "Observations";
          this.loadImageAnalysisResult(id, image);
          this.loadCameraInfo(id, address, image);
          map.flyTo({
            center: e.features[0].geometry.coordinates,
            zoom: 18,
            bearing: map.getBearing() + 45,
            pitch: 60
          });
        });
        map.on("mouseenter", "cameras", () => {
          map.getCanvas().style.cursor = "pointer";
        });
        map.on("mouseleave", "cameras", () => {
          map.getCanvas().style.cursor = "";
        });
      });
    });
    this.map = map;

    this.loadSummaryInfo(this.temporalWindow);

    this.loadSummaryMap(this.temporalWindow);
  }
};
</script>
<style>
@import "../node_modules/mapbox-gl/dist/mapbox-gl.css";
@import "../node_modules/@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";

body {
  margin: 0;
  padding: 0;
}
#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
}
.v-overlay {
  z-index: 1 !important;
}
.summary-info,
.camera-info {
  z-index: 2 !important;
}
.app-map-legend {
  z-index: 2;
  padding: 10px;
}
.app-map-legend-item {
  width: 20px;
  height: 20px;
  float: left;
}
.app-map-legend-title {
  padding: 10px;
  font-size: 12px;
}
.app-logo img {
  width: 25px !important;
  position: absolute;
  bottom: 10px;
  left: 100px;
}
</style>