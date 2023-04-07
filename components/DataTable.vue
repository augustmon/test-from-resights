<template lang="pug">


v-data-table(
    :headers="headers"
    :items="items"
    item-key="email"
    dense
    @update:page="(page) => $emit('new-page-number', page)"
    @update:items-per-page="(rows) => $emit('new-page-size', rows)"
    
    :loading="loading"

    :items-per-page="pageSize"
    :page="currentPage"

    :server-items-length="1000"
    :footer-props="{itemsPerPageOptions: [5,10,15,25] }"
  ).elevation-1.mt-10
    //- v-data-footer(

    //- )
  

</template>

<script>
export default {
  props: ['headers', 'items', 'currentPage', 'pageSize', 'loading', 'options'],
  data() {
    return {  
  }
},
  emits: ['new-page-number', 'new-page-size'],
  methods: {
  }
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
</style>
