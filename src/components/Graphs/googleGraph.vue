<template>
    <GChart
            :type="type"
            :data="data"
            :options="options"
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
            'options',
            'type'
        ],
    }
</script>
