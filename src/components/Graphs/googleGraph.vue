<template>
    <GChart
            :type="type"
            :data="data"
            :options="chartOptions"
    />
</template>

<script>
    import {GChart} from 'vue-google-charts'

    export default {
        data() {
            return {
            }
        },
        components: {
            GChart,
        },
        props: [
            'data',
            'chartOptions',
            'type'
        ],
    }
</script>
