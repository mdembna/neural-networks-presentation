<template>
  <div>
    <p v-if="trainingInProgress">Training linear model... </p>
    <template v-else>
      <input @input="predict" type="number">
    </template>
  </div>
</template>

<script>
import * as tf from "@tensorflow/tfjs";

export default {
  props: {
    dataset: Array,
    labels: Array
  },
  data() {
    return {
      trainingInProgress: false,
      prediction: null,
      linearModel: null,
    }
  },
  async mounted() {
    this.trainingInProgress = true;

    await this.trainNewModel();

    this.trainingInProgress = false;
  },
  methods: {
    async trainNewModel() {

      // Model for linear regression
      this.linearModel = tf.sequential();

      // Add layers
      this.linearModel.add(tf.layers.dense({ units: 1, inputShape: [ 1 ]}));

      //Prepare the model for training: specify the loss and optimizer
      // loss (also called cost) - the lower the value, the better model
      // optimizer: sgd - stochastic gradient descent (basically shuffles dataset and splits it into smaller chunks, optimizes calculations) - drunk person going downhill analogy

      this.linearModel.compile({ loss: 'meanSquaredError', optimizer: 'sgd'});

      // Training model - feed data in the form of a tensor
      // TODO: tensor def (and its dimensions)
      // supervised training:
      // x - training set
      // y - label for the data

      const xs = tf.tensor1d(this.dataset);
      const ys = tf.tensor1d(this.labels);

      // Train
      // In-browser models shouldn't deal with datasets bigger than 30mb

      await this.linearModel.fit(xs, ys);

      console.log('trained!');

    },
    predict(value) {
      const numericInput = parseFloat(value.target.value);
      if (isNaN(numericInput)) return;

      const output = this.linearModel.predict(tf.tensor2d([numericInput], [1, 1]))

      this.prediction = Array.from(output.dataSync())[0];

      console.log(this.prediction);

      this.$emit('predict', numericInput, this.prediction);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
