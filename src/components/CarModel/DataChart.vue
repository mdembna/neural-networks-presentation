<template>
  <div class="chart-wrapper">
    <canvas ref="chart" width="800" height="400"></canvas>
  </div>
</template>

<script>
import {Chart, registerables} from "chart.js";
import debounce from "lodash.debounce";

export default {
  props: {
    dataset: Array,
    predictedPoints: Array,
  },
  data() {
    return {
      chart: null
    }
  },
  computed: {
    chartData() {
      return {
        datasets: [
          {
            label: 'Prediction',
            data: this.predictedPoints,
            backgroundColor: [
              'rgba(255, 104, 99, 0.4)',
            ],
            borderColor: [
              'rgba(255, 104, 99, 1)',
            ],
            borderWidth: 1,
          },
          {
            label: 'Training data',
            data: this.dataset.map((value) => ({ x: value.horsepower, y: value.mpg})),
            backgroundColor: [
              'rgba(99, 104, 255, 0.4)',
            ],
            borderColor: [
              'rgba(99, 104, 255, 1)',
            ],
            borderWidth: 1
          },
        ]
      }
    },
  },
  watch: {
    dataset() {
      this.initChart();
    },
    predictedPoints() {
      this.debouncedUpdate(this);
    }
  },
  mounted() {
    Chart.register(...registerables);
  },
  methods: {
    debouncedUpdate: debounce((_) => {
      _.chart.data = _.chartData;
      _.chart.update();
    }, 1000),
    initChart() {
      this.chart = new Chart(this.$refs.chart.getContext('2d'), {
        type: 'scatter',
        data: this.chartData,
        options: {
          scales: {
            x: {
              type: 'linear',
              position: 'bottom',
              title: {
                display: true,
                text: 'horsepower'
              }
            },
            y: {
              type: 'linear',
              position: 'left',
              title: {
                display: true,
                text: 'miles per gallon'
              }
            }
          }
        }
      });
    }
  }
}
</script>

<style scoped>
.chart-wrapper {
  width: 800px;
  margin: auto;
}
</style>
