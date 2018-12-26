<template>
  <v-app dark>
    <v-container grid-list-md>
      <v-layout row wrap>
        <v-flex sm6 xs12>
          <v-card class="grey darken-2 pa-2">
            <v-card-title>
              <h4 class="display-1 blue--text text--darken-2">Current Accuracy</h4>
            </v-card-title>
            <div class="text-xs-center py-2">
              <v-progress-circular
                :size="70"
                :width="7"
                color="green"
                indeterminate
                class="tt"
                v-if="loading"
              ></v-progress-circular>
              <h3 class="display-3 green--text text--darken-1" v-else>{{accuracy}}%</h3>
            </div>
          </v-card>
        </v-flex>
        <v-flex sm6 xs12>
          <v-card class="grey darken-2 pa-2">
            <v-card-title>
              <h4 class="display-1 blue--text text--darken-2">HMM Stats</h4>
            </v-card-title>
          </v-card>
        </v-flex>
        <v-flex sm12>
          <v-card class="grey darken-2 pa-2">
            <v-text-field label="Input 1" v-model="input1"></v-text-field>
            <v-text-field label="Input 2" v-model="input2"></v-text-field>
            <v-btn @click="predict" class="blue accent-3">Predict</v-btn>
          </v-card>
        </v-flex>
        <v-flex sm12>
          <predictionTable :headers="headers" :genomes="items" />
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import * as tf from "@tensorflow/tfjs";
import dna from "./assets/training.json";
import dnaTesting from "./assets/testing.json";
import input from "./assets/input.json";
import output from "./assets/output.json";
import Spinner from "./components/UI/spinner";
import PredictionTable from "./components/UI/predictionTable";

export default {
  name: "App",
  components: {
    spinner: Spinner
  },
  data() {
    return {
      trainingData: [],
      outputData: [],
      testingData: [],
      loading: true,
      input1: null,
      input2: null,
      accuracy: null,
      headers: [
        { text: 'Genome', align: 'left', sortable: false, value: 'genome' },
        { text: 'Prediction', align: 'left', value: 'prediction', sortable: true }
      ],
      items: []
    };
  },
  methods: {
    train() {
      const model = (this.model = tf.sequential());

      model.add(
        tf.layers.dense({
          inputShape: [2],
          activation: "sigmoid",
          units: 3
        })
      );

      model.add(
        tf.layers.dense({
          inputShape: [3],
          activation: "sigmoid",
          units: 2
        })
      );

      model.add(
        tf.layers.dense({
          activation: "sigmoid",
          units: 2
        })
      );

      model.compile({
        loss: "meanSquaredError",
        optimizer: tf.train.adam(0.06)
      });

      model
        .fit(this.trainingData, this.outputData, { epochs: 100 })
        .then(history => {
          model.predict(this.testingData).print();
          const h =
            (1 - history.history.loss[history.history.loss.length - 1]) * 100;
          this.accuracy = Math.round(h * 100) / 100;
          this.loading = false;
        });
    },
    predict() {
      const x1 = input.find(value => {
        if (value.input === this.input1) {
          return value;
        }
      });

      const x2 = input.find(value => {
        if (value.input === this.input1) {
          return value;
        }
      });

      const predictedValue = this.model.predict(tf.tensor2d([[x1.id, x2.id]]));
      // console.log(predictedValue);
      predictedValue.data().then(d => {
        d.forEach((element, index) => {
          output[index] = { ...output[index], prediction: element };
        });
        this.items = output;
      });
    }
  },
  created() {
    this.trainingData = tf.tensor2d(dna.map(item => [item.x1, item.x2]));
    this.outputData = tf.tensor2d(
      dna.map(item => [
        item.y === "escherichia coli" ? 1 : 0,
        item.y === "pseudomonas_aeriginosa pao1" ? 1 : 0
      ])
    );
    this.testingData = tf.tensor2d(dnaTesting.map(item => [item.x1, item.x2]));
  },
  mounted() {
    this.train();
    // this.predict();
  },
  components: {
    'predictionTable': PredictionTable
  }
};
</script>

<style>
.bg {
  background-color: black;
}

.tt {
  width: 30px;
}
</style>

