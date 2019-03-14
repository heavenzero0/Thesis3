<template>
  <v-data-table :headers="headers" :items="genomes" class="elevation-1"  :rows-per-page-items="rowsPerPageItems" dark>
    <template slot="items" slot-scope="props">
      <td>{{ props.item.genome }}</td>
      <td>{{ props.item.prediction }}</td>
    </template>
  </v-data-table>
</template>


<script>
export default {
  data() {
    return{
rowsPerPageItems: [2, 5, 10, 15,]

    }
  },
  props: [
    'headers',
    'genomes'
  ]
};
</script>

