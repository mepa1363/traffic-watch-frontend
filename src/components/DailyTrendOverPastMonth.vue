<template>
  <apexchart ref="chart" type="line" height="225" :options="chartOptions" :series="series" />
</template>

<script>
import axios from "axios";
import moment from "moment";

export default {
  props: ["cameraId"],
  data: () => ({
    vehicles: [],
    pedestrians: [],
    bikes: [],
    allVehicles: [],
    allPedestrians: [],
    allBikes: [],
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
      colors: ["#FF4560", "#0090FF", "#00E396"],
      dataLabels: {
        enabled: false
      },
      stroke: {
        width: [2, 2, 2, 2, 2, 2],
        curve: "smooth",
        dashArray: [0, 0, 0, 2, 2, 2]
      },
      grid: {
        borderColor: "#f1f1f1",
        padding: {
          top: 30
        }
      },
      xaxis: {
        type: "datetime",
        labels: {
          formatter: (value, timestamp, index) => {
            return moment(new Date(timestamp)).format("MMM DD");
          }
        }
      },
      tooltip: {
        x: {
          format: "dd MMM yyyy"
        }
      },
      legend: {
        position: "top",
        horizontalAlign: "center",
        floating: true,
        itemMargin: {
          horizontal: 2
        }
      }
    }
  }),
  computed: {
    series() {
      return [
        {
          name: "Vehicles",
          data: this.vehicles
        },
        {
          name: "Bikes",
          data: this.bikes
        },
        {
          name: "Pedestrians",
          data: this.pedestrians
        },
        {
          name: "Average (V)",
          data: this.allVehicles
        },
        {
          name: "Average (B)",
          data: this.allBikes
        },
        {
          name: "Average (P)",
          data: this.allPedestrians
        }
      ];
    }
  },
  mounted() {
    axios
      .get(
        `${process.env.VUE_APP_API}/camera/${this.cameraId}/chart/past-month`
      )
      .then(response => {
        this.vehicles = response.data.vehicles;
        this.pedestrians = response.data.pedestrians;
        this.bikes = response.data.bikes;
      })
      .catch(error => {
        console.log(error);
      });
    axios
      .get(`${process.env.VUE_APP_API}/camera/all/chart/past-month`)
      .then(response => {
        this.allVehicles = response.data.vehicles;
        this.allPedestrians = response.data.pedestrians;
        this.allBikes = response.data.bikes;
      })
      .catch(error => {
        console.log(error);
      });
  },
  updated() {
    this.$refs.chart.toggleSeries("Average (V)");
    this.$refs.chart.toggleSeries("Average (P)");
    this.$refs.chart.toggleSeries("Average (B)");
  }
};
</script>
