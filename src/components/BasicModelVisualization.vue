<template>
  <div>
    <Chart :labels="labels" :dataset="dataset" :input="input" :output="calculatedOutput" :expected-output="expectedOutput"/>

    <p>expected output: {{ expectedOutput }}</p>
    <p>output: {{ calculatedOutput }}</p>
    <p>input: {{ input }}</p>
    <LinearModel :labels="labels" :dataset="dataset" @predict="setPredictedPoint"/>
  </div>
</template>

<script>
import Chart from "@/components/Chart";
import LinearModel from "@/components/LinearModel";

export default {
  components: { LinearModel, Chart },
  data() {
    return {
      // larger size = better accuracy
      datasetSize: 20000,
      dataset: [],
      labels: [],
      calculatedOutput: 0,
      expectedOutput: 0,
      input: 0,
    }
  },
  async mounted() {
    this.generateRandomDataset();
  },
  methods: {
    calculateValue(value) {
      return value*2;
    },
    generateRandomDataset() {
      this.dataset = new Array(this.datasetSize).fill(1).map((_, i) => i);
      this.labels = this.dataset.map(this.calculateValue);
    },
    setPredictedPoint(input, prediction) {
      this.input = input;
      this.expectedOutput = this.calculateValue(input);
      this.calculatedOutput = prediction;
    }
  }
}
</script>