<template>
  <div>
    <DataChart :dataset="dataset" :predicted-points="predictedPoints"/>
    <Model v-if="dataset.length" :dataset="dataset" @predict="predictedPoints = $event"/>
  </div>
</template>

<script>
import axios from 'axios';
import DataChart from "@/components/CarModel/DataChart";
import Model from "@/components/CarModel/Model";

export default {
  components: {Model, DataChart},
  data() {
    return {
      dataset: [],
      predictedPoints: []
    }
  },
  async mounted() {
    await this.loadData();
  },
  methods: {
    async loadData() {
      try {
        const {data} = await axios.get('https://storage.googleapis.com/tfjs-tutorials/carsData.json');

        this.dataset = data.map(car => ({
          mpg: car.Miles_per_Gallon,
          horsepower: car.Horsepower,
        }))
            .filter(car => (car.mpg != null && car.horsepower != null));
      } catch (err) {
        console.log(err);

        this.dataset = [];
      }
    }
  }
}
</script>