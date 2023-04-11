<template lang="pug">

v-data-table(
    :headers="headers"
    :items="items"
    item-key="email"
    dense
    @update:page="(page) => $emit('new-page-number', page)"
    @update:items-per-page="(rows) => $emit('new-page-size', rows)"
    @update:sort-by="(category) => $emit('new-sort-by', category)"
    @update:sort-desc="(bool) => $emit('new-sort-desc', bool)"
    :loading="loading"
    :items-per-page="pageSize"
    :page="currentPage"
    :server-items-length="serverItemsLength"
    :footer-props="{itemsPerPageOptions: [5,10,20,30] }"
    :sort-by="sortBy"
    :sort-desc="sortDesc"
  ).elevation-1.mt-0
    template(
      v-slot:footer.prepend)
      v-btn(
        @click="() => $emit('refresh')" 
        icon 
        color='gray'
        :loading="loading"
        
        )
        v-icon mdi-cached
    
  

</template>

<script>
export default {
  props: ['headers', 'items', 'currentPage', 'pageSize', 'loading', 'options', 'serverItemsLength', 'sortBy','sortDesc', 'refresh'],
  data() {
    return {  
  }
},
  emits: ['new-page-number', 'new-page-size', 'new-sort-by', 'new-sort-desc'],
  methods: {
  }
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
    
</style>
