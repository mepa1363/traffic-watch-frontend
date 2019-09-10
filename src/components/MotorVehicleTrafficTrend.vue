<template>
  <apexchart type="line" height="200" :options="chartOptions" :series="series" />
</template>


<script>
import axios from "axios";
export default {
  props: ["cameraId"],
  data: () => ({
    observation: [],
    prediction: [],
    chartOptions: {
      chart: {
        shadow: {
          enabled: true,
          color: "#000",
          top: 18,
          left: 7,
          blur: 10,
          opacity: 1
        },
        toolbar: {
          show: false
        },
        zoom: {
          enabled: false
        }
      },
      colors: ["#FF4560", "#0090FF"],
      dataLabels: {
        enabled: false
      },
      stroke: {
        width: [2, 2],
        curve: "smooth",
        dashArray: [0, 2]
      },
      grid: {
        borderColor: "#f1f1f1"
      },
      xaxis: {
        type: "datetime"
      },
      tooltip: {
        x: {
          format: "HH:MM"
        }
      },
      legend: {
        position: "top",
        horizontalAlign: "center",
        floating: true
      }
    }
  }),
  computed: {
    series() {
      return [
        {
          name: "Observation",
          data: this.observation
        },
        {
          name: "Prediction",
          data: this.prediction
        }
      ];
    }
  },
  mounted() {
    axios
      .get(`${process.env.VUE_APP_API}/camera/${this.cameraId}/prediction`)
      .then(response => {
        this.observation = response.data.observation;
        this.prediction = response.data.prediction;
      })
      .catch(error => {
        console.log(error);
      });
  }
};
</script>
