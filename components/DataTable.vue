<template lang="pug">
v-data-table(
    :headers="headers"
    :items="items"
    item-key="email"
    dense
    @update:page="(page) => emitPageNumber(page)"
    @update:items-per-page='(rows) => emitPageSize(rows)'
  ).elevation-1.mt-10
</template>

<script>
export default {
  props: ['headers', 'items', 'page', 'items-per-page'],
  data() {
    return {   
      currentPage: 'page'
    }
  },
  emits: ['new-page-number', 'newRows'],
  methods: {
    emitPageNumber(page) {
      this.$emit('new-page-number', page);
    },
    emitPageSize(rows) {
      this.$emit('new-page-size', rows)
    }
  }
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
</style>
