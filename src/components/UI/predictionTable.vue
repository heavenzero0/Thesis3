<template>
  <v-data-table :headers="headers" :items="genomes" class="elevation-1">
    <template slot="items" slot-scope="props">
      <td>{{ props.item.genome }}</td>
      <td>{{ props.item.prediction }}</td>
    </template>
  </v-data-table>
</template>


<script>
export default {
  data() {
    return{}
  },
  props: [
    'headers',
    'genomes'
  ]
};
</script>

