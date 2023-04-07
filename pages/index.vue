

<template lang="pug">
v-container
    v-row 
      v-col(
        md=3)
        v-text-field( 
          type='text' 
          v-model="searchQuery" 
          placeholder='Search for a record...'
          append-icon="mdi-magnify"
          single-line
          hide-details)
    v-row
      v-col(cols)
        DataTable(
          v-if="items.length"
          :headers="headers"
          :items="searchQuery.length < 3 ? items : filteredEntries"
          @new-page-number="(page) => updatePageNumber(page)"
          @new-page-size="(rows) => updatePageSize(rows)"
          :pageSize="pageSize"
        )
        v-progress-circular(
          v-else
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto
    <p> {{ pageNumber }}</p>
    <v-pagination v-model="page" :length="100" :total-visible="10"> </v-pagination>
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

export default {
  components: {
    DataTable
  },
  data() {
    return {
      sales,
      items: [],
      headers: [
        { text: 'First Name', value: 'user.first_name', align: 'start' },
        { text: 'Last Name', value: 'user.last_name'},
        { text: 'Email', value: 'email' },
        { text: 'Gender', value: 'gender' },
        { text: 'Year', value: 'year' },
        { text: 'Sales', value: 'sales' },
        { text: 'Country', value: 'country' },
      ],
      pageNumber: 1,
      pageSize: 10, 
      searchQuery: '',
      fetchedEntries: 20

    }
  },

  computed: {
    filteredEntries() {
      function getObjectProps(obj) {
        let allProps = ""
        for (const prop in obj) {
          if (typeof obj[prop] === 'object') {
            allProps = allProps.concat(getObjectProps(obj[prop]))
          }
          else {
             allProps = allProps.concat(" ", obj[prop])
          }
        }
        return allProps
      }
      
      return this.items.filter((item) => getObjectProps(item).toLowerCase().includes(this.searchQuery))
    },   
  },

  watch: {
    async pageNumber(number) {
      if (this.fetchedEntries < (this.pageNumber*this.pageSize) + this.pageSize ) {
        let pageLoad = await this.fetchData(Math.ceil(this.fetchedEntries)/this.pageSize, this.pageSize)
        this.items = [...this.items, ...pageLoad]
        this.fetchedEntries += pageLoad.length
      }
    }
  },

  async created() {
    this.items = await this.fetchData(0, this.pageSize*2)
  },

  methods: {
    async fetchData(page, size) {
      const start = page * size
      const end = start + size
      await this.delay(3000)
      return await sales.results.slice(start, end)
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    updatePageNumber(number) {
      this.pageNumber = number
    },
    updatePageSize(rows) {
      this.pageSize = rows
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>