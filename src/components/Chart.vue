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
    labels: Array,
    input: Number,
    expectedOutput: Number,
    output: Number
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
            data: [{ x: this.input, y: this.expectedOutput}],
            backgroundColor: ['green'],
            borderColor: ['green'],
            borderWidth: 10,
            borderRadius: 100
          },
          {
            label: 'Output',
            data: [{ x: this.input, y: this.output}],
            backgroundColor: ['yellow'],
            borderColor: ['yellow'],
            borderWidth: 10,
            borderRadius: 100
          },
          {
            label: 'Dataset',
            data: this.dataset.map((value, i) => ({ x: value, y: this.labels[i]})),
            backgroundColor: [
              'rgba(255, 104, 99, 0.4)',
            ],
            borderColor: [
              'rgba(255, 104, 99, 1)',
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
    input() {
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
              position: 'bottom'
            },
            y: {
              type: 'linear',
              position: 'right'
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
