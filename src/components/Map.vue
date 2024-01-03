<template>
  <div id="map"></div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import { EventBus } from "../assets/eventBus";

export default {
  name: "MapComponent",
  props: {
    countries: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      map: null,
    };
  },
  watch: {
    countries: {
      handler: "updateCirclesOnMap",
      immediate: true,
    },
  },
  mounted() {
    this.initializeMap();
    EventBus.$on("focus-country", this.focusMapOnCountry);
  },
  methods: {
    initializeMap() {
      mapboxgl.accessToken =
        "pk.eyJ1Ijoibmlrb2xhc3QiLCJhIjoiY2xxcXhnNHI0M20yOTJrbzRtcGRnZDNlcyJ9.ZYWTLGL3Nf2F9AhEdaKlsQ";
      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/streets-v11",
        center: [0, 0],
        zoom: 2,
      });

      this.loadCSVData();
    },
    focusMapOnCountry(country) {
      if (this.map && country.latlng) {
        this.map.flyTo({
          center: [country.latlng[1], country.latlng[0]],
          zoom: 5,
          essential: true,
        });
      }
    },
    async loadCSVData() {
      try {
        const response = await fetch("/vessel-tracks-data.csv");
        const text = await response.text();
        const data = this.parseCSV(text);
        const geojsonData = this.convertToGeoJSON(data);
        console.log(geojsonData);
        this.addVesselTracksToMap(geojsonData);
        this.addRedCirclesToMap(); // Add red circles after loading the CSV data
      } catch (error) {
        console.error("Error loading CSV data:", error);
      }
    },
    parseCSV(csvText) {
      const lines = csvText.split("\n");
      const result = [];
      const headers = lines[0].split(",").map((header) => header.trim());

      for (let i = 1; i < lines.length; i++) {
        if (!lines[i]) continue;
        const obj = {};
        const currentline = lines[i].split(",");

        for (let j = 0; j < headers.length; j++) {
          obj[headers[j]] = currentline[j].trim();
        }

        result.push(obj);
      }

      return result;
    },
    convertToGeoJSON(csvData) {
      const groupedData = csvData.reduce((acc, item) => {
        const vesselId = item["vessel_id"];
        if (!acc[vesselId]) {
          acc[vesselId] = [];
        }
        acc[vesselId].push([
          parseFloat(item["longitude"]),
          parseFloat(item["latitude"]),
        ]);
        return acc;
      }, {});

      const features = Object.keys(groupedData).map((vesselId) => {
        return {
          type: "Feature",
          properties: {
            vessel_id: vesselId,
          },
          geometry: {
            type: "LineString",
            coordinates: groupedData[vesselId],
          },
        };
      });

      return {
        type: "FeatureCollection",
        features: features,
      };
    },
    addVesselTracksToMap(geojsonData) {
      this.map.on("load", () => {
        this.map.addSource("vessel-tracks", {
          type: "geojson",
          data: geojsonData,
        });

        this.map.addLayer({
          id: "vessel-tracks-layer",
          type: "line",
          source: "vessel-tracks",
          layout: {
            "line-join": "round",
            "line-cap": "round",
          },
          paint: {
            "line-width": 1,
            "line-color": "#398df7",
            "line-opacity": 0.2,
            "line-dasharray": [2, 4],
          },
        });
      });
    },
    addRedCirclesToMap() {
      this.map.addSource("country-circles", {
        type: "geojson",
        data: {
          type: "FeatureCollection",
          features: this.countries.map((country) => ({
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [country.latlng[1], country.latlng[0]],
            },
            properties: {
              title: country.name.common,
            },
          })),
        },
      });

      this.map.addLayer({
        id: "country-circles-layer",
        type: "circle",
        source: "country-circles",
        paint: {
          "circle-radius": 5,
          "circle-color": "#ff5349",
        },
      });
    },
    updateCirclesOnMap(newCountries) {
      if (this.map.getLayer("country-circles-layer")) {
        this.map.removeLayer("country-circles-layer");
      }

      if (this.map.getSource("country-circles")) {
        this.map.removeSource("country-circles");
      }

      this.addRedCirclesToMap(newCountries);
    },
  },
};
</script>
