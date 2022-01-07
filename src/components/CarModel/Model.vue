<template>
  <div>
    <div v-if="isTraining">Training model in progress...</div>
    <div v-else>
      <button class="predictions-button" :class="{ predicting }" @click="predict">Make predictions</button>
    </div>
  </div>
</template>

<script>
import * as tf from "@tensorflow/tfjs";

  export default {
    props: {
      dataset: Array,
    },
    data() {
      return {
        isTraining: true,
        model: null,
        predicting: false,
        normalizedData: {}
      }
    },
    async mounted() {
      // Create a model
      this.model = this.createModel();

      // Convert the data to a form we can use for training.
      this.normalizedData = this.prepareDataForTraining(this.dataset);

      // Train the model
      await this.trainModel(this.model, this.normalizedData.inputs, this.normalizedData.labels);

      this.isTraining = false;
    },
    methods: {
      createModel() {
        // Create a sequential model
        const model = tf.sequential();

        // Add a single input layer
        // This adds an input layer to our network, which is automatically connected to a dense layer with one hidden unit. A dense layer is a type of layer that multiplies its inputs by a matrix (called weights) and then adds a number (called the bias) to the result. As this is the first layer of the network, we need to define our inputShape. The inputShape is [1] because we have 1 number as our input (the horsepower of a given car).
        // units sets how big the weight matrix will be in the layer. By setting it to 1 here we are saying there will be 1 weight for each of the input features of the data.

        model.add(tf.layers.dense({inputShape: [1], units: 1, useBias: true}));

        // Add an output layer
        // It creates our output layer. We set units to 1 because we want to output 1 number.
        model.add(tf.layers.dense({units: 1, useBias: true}));

        return model;
      },
      prepareDataForTraining(data) {
        // Wrapping these calculations in a tidy will dispose any
        // intermediate tensors.

        return tf.tidy(() => {
          // Step 1. Shuffle the data
          tf.util.shuffle(data);

          // Step 2. Convert data to Tensor
          const inputs = data.map(d => d.horsepower)
          const labels = data.map(d => d.mpg);

          const inputTensor = tf.tensor2d(inputs, [inputs.length, 1]);
          const labelTensor = tf.tensor2d(labels, [labels.length, 1]);

          //Step 3. Normalize the data to the range 0 - 1 using min-max scaling
          const inputMax = inputTensor.max();
          const inputMin = inputTensor.min();
          const labelMax = labelTensor.max();
          const labelMin = labelTensor.min();

          const normalizedInputs = inputTensor.sub(inputMin).div(inputMax.sub(inputMin));
          const normalizedLabels = labelTensor.sub(labelMin).div(labelMax.sub(labelMin));

          return {
            inputs: normalizedInputs,
            labels: normalizedLabels,
            // Return the min/max bounds so we can use them later.
            inputMax,
            inputMin,
            labelMax,
            labelMin,
          }
        });
      },
      async trainModel(model, inputs, labels) {
        // Prepare the model for training.
        model.compile({
          optimizer: tf.train.adam(),
          loss: tf.losses.meanSquaredError,
          metrics: ['mse'],
        });

        const batchSize = 32;
        const epochs = 50;

        return await model.fit(inputs, labels, {
          batchSize,
          epochs,
          shuffle: true,
          // callbacks: tfvis.show.fitCallbacks(
          //     { name: 'Training Performance' },
          //     ['loss', 'mse'],
          //     { height: 200, callbacks: ['onEpochEnd'] }
          // )
        });
      },
      predict() {
        this.predicting = true;
        const {inputMax, inputMin, labelMin, labelMax} = this.normalizedData;

        // Generate predictions for a uniform range of numbers between 0 and 1;
        // We un-normalize the data by doing the inverse of the min-max scaling
        // that we did earlier.
        const [xs, preds] = tf.tidy(() => {

          const xs = tf.linspace(0, 1, 100);
          const preds = this.model.predict(xs.reshape([100, 1]));

          const unNormXs = xs
              .mul(inputMax.sub(inputMin))
              .add(inputMin);

          const unNormPreds = preds
              .mul(labelMax.sub(labelMin))
              .add(labelMin);

          // Un-normalize the data
          return [unNormXs.dataSync(), unNormPreds.dataSync()];
        });


        const predictedPoints = Array.from(xs).map((val, i) => {
          return {x: val, y: preds[i]}
        });

        this.$emit('predict', predictedPoints)

        this.predicting = false;
      }
    }
  }
</script>
<style scoped>
.predictions-button {
  border: 1px solid rgba(0, 0, 0, .2);
  border-radius: 5px;
  padding: 10px;
  margin: 50px 0;
  cursor: pointer;
  background: transparent;
}

.predicting {
  pointer-events: none;
  color: rgba(0, 0, 0, .2);
  cursor: not-allowed;
}
</style>