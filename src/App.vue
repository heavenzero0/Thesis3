<template>
    <v-app light>
        <v-container grid-list-md>
            <v-layout row wrap>
                <v-flex sm4>
                         <div class="text-xs-center py-2">
                            <v-progress-circular
                                    :size="90"
                                    :width="7"
                                    color="green"
                                    indeterminate
                                    class="tt"
                                    v-if="loading"
                            ></v-progress-circular>
                            <v-card-title v-if="!loading" mt-4>
                                <h6 class="display-3 green--text text--darken-1" >{{accuracy}}%</h6>
                                <h6 class="display-2 green--text text--darken-1 font-weight-thin">Accuracy</h6>
                                <h6 class="display-3 green--text text--darken-1" >Training </h6>
                                <h6 class="display-2 green--text text--darken-1 font-weight-thin">{{training}} seconds</h6>
                                
                            </v-card-title>
                           </div>
                </v-flex>
                <v-flex sm8>
                    <div style="height:400px" v-if="loading"></div>
                    <v-card  v-else height="400px">
                        <GChart v-if="!loading" :data="data" :options="chartOptions" type="LineChart" height="350px" />
                    </v-card>
               
                </v-flex>
                <!-- <v-flex sm4>
                <v-card class="green lighten-3 pa-2" height="215px">
                        <v-text-field label="Input 1" v-model="input1"></v-text-field>
                        <v-text-field label="Input 2" v-model="input2"></v-text-field>
                        <v-btn @click="predict" class="green darken-2">Predict</v-btn>
                         <v-btn class="green darken-2" @click="automate">automate</v-btn> 
                      
                    </v-card> -->
<!-- 
                 </v-flex>
                 <div v-if="loading"></div>
                <v-flex v-else dark>
                      <v-card>
                         <predictionTable :headers="headers" :genomes="items"/>
                </v-card>
                </v-flex>

 -->
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
    // import Spinner from "./components/UI/spinner";
    import GChart from "./components/Graphs/googleGraph";
    import PredictionTable from "./components/UI/predictionTable";

    export default {
        name: "App",
        data() {
            return {
                trainingData: [],
                outputData: [],
                testingData: [],
                loading: true,
                training : 0,
                input1: null,
                input2: null,
                accuracy: null,
                headers: [
                    {text: 'Genome', align: 'left', sortable: false, value: 'genome'},
                    {text: 'Prediction', align: 'left', value: 'prediction', sortable: true}
                ],
                items: [],
                data: [["Iterations", "Accuracy"]], //SET THE DATA TO PASS IN THE GRAPH
                chartOptions: { //OPTIONS FOR THE CHARTS
                    title: 'DNA SEQUENCE',
                    vAxis: {
                        minValue: 0,
                        maxValue: 100,
                        textStyle: {bold: true},
                        title: 'Percentage'
                    },
                    hAxis: {
                        textStyle: {
                            fontName: 'Tahoma',
                            bold: true,
                            fontSize: 15
                        },
                        title: 'Iterations'
                    },
                    height: 400,
                    legend: {position: 'top', maxlines: 3,},
                    colors: ['#5870cb'],
                    tooltip: {isHtml: true},
                    chartArea: {width: '80%',},
                },
                value: []
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
                        units: 5
                    })
                );
                model.add(
                    tf.layers.dense({
                        activation: "sigmoid",
                        units: 5
                    })
                );
                model.compile({
                    loss: "meanSquaredError",
                    optimizer: tf.train.adam(0.06)
                });
                const beforeTrain = Date.now();
                model
                    .fit(this.trainingData, this.outputData, {epochs: 100})
                    .then(history => {
                        model.predict(this.testingData).print();
                        
                        const h =
                            (1 - history.history.loss[history.history.loss.length - 1]) * 100;
                        this.accuracy = (Math.round(h * 100) / 100 );

                        //CHARTS DATA ARE ALL SET HERE
                        history.history.loss.forEach((val, index) => {
                            const accuracy = ((1 - val) * 100 );
                            const train = [index + 1, Math.round(accuracy * 100) / 100];
                            this.value = [...this.value, Math.round(accuracy * 100) / 100];
                            this.data = [...this.data, train];
                        });
                        //CONSOLE LOG this.data TO KNOW THE VALUE INSIDE
                        this.loading = false;
                       this.training = ((Date.now()-beforeTrain)/1000);
                    });
            },
            predict() {
                this.items = [];
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
                        output[index] = {...output[index], prediction: (1 - +element ) *100};
                    });
                    this.items = output;
                });
            },
            automate() {
                let arrr = [];
                for (let i = 901; i <= 1000; i++) {
                    const dd = {
                        "x1": i,
                        "x2": i + 1,
                        "y": "Vibrio cholerae O1"
                    };
                    arrr = [...arrr, dd];
                }
                console.log(arrr);
            }
        },
        created() {
            this.trainingData = tf.tensor2d(dna.map(item => [item.x1, item.x2]));
            this.outputData = tf.tensor2d(
                dna.map(item => [
                    item.y === "escherichia coli" ? 1 : 0,
                    item.y === "pseudomonas_aeriginosa pao1" ? 1 : 0,
                     item.y === "Caulobacter vibrioides CB15" ? 1 : 0,
                    item.y === "Vibrio cholerae O1" ? 1 : 0,
                     item.y === "Sinorhizobium meliloti 1021" ? 1 : 0
                ])
            );
            this.testingData = tf.tensor2d(dnaTesting.map(item => [item.x1, item.x2]));
        },
        mounted() {
          this.train();
            //this.automate();
            // this.predict();
        },
        components: {
            'predictionTable': PredictionTable,
            'GChart': GChart
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

 