<template lang="pug">
v-container
  v-row(
    align="start")
    v-col(
      md=4)
      v-text-field( 
        type='text' 
        v-model="searchQuery" 
        placeholder='Search for a record...'
        :append-icon='searchQuery ? "mdi-close" : "mdi-magnify"'
        single-line
        @click:append = 'searchQuery ? searchQuery = "" : refresh'
        hide-details)
    v-col(
      md=2)
      v-select(
        :items="[headers[0],headers[1],headers[2],headers[3],headers[6]]"
        label="Search in column"
        clearable
        @change='(event) => activeFilter = event' 
        @click:clear='activeFilter = null'
        loader-height=1
        hide-details
      )
     
  v-row
    v-col(cols)
      DataTable(
        v-if="items.length || loading"
        :headers="headers"
        :items="searchQuery.length < 3 ? items : searchItems"
        :serverItemsLength="!searchItems.length ? 1000 : totalSearchItems"
        @new-page-number="(page) => updateCurrentPage(page)"
        @new-page-size="(rows) => updatePageSize(rows)"
        @new-sort-by="(category) => sortBy = category"
        @new-sort-desc="(bool) => sortDesc = bool"
        :currentPage.sync="currentPage"
        :pageSize.sync="pageSize"
        :loading="loading"
        :sortDesc.sync=sortDesc 
        :sortBy.sync=sortBy 
        @refresh="() => refresh()"
        )
      v-progress-circular(
        v-else
        width="2"
        color="rs__primary"
        indeterminate
      ).mx-auto
      

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
      searchItems: [],
      loading: true,
      options: {},
      headers: [
        { text: 'First Name', value: 'user.first_name', align: 'start' },
        { text: 'Last Name', value: 'user.last_name'},
        { text: 'Email', value: 'email' },
        { text: 'Gender', value: 'gender' },
        { text: 'Year', value: 'year' },
        { text: 'Sales', value: 'sales' },
        { text: 'Country', value: 'country' },
      ],
      currentPage: 1,
      pageSize: 10, 
      searchQuery: '',
      activeFilter: null,
      totalSearchItems: 0,
      sortBy: '',
      sortDesc: false
    }
  },

  watch: {
    searchQuery() {
      if (this.searchQuery.length > 2) 
      {
        this.currentPage = 1
        this.performAPISearch()
      }
      else if (this.searchQuery.length == 0) {
        this.searchItems = []
        this.currentPage = 1
      }
    },

    activeFilter() {
      if (this.searchQuery.length > 2) {
        this.performAPISearch()
          }
        },

      sortBy() {
        this.loading = true
        this.fetchData(this.currentPage, this.pageSize).then(data => {
          this.items = data; this.loading = false});
      },
      sortDesc() {
        this.loading = true
        this.fetchData(this.currentPage, this.pageSize).then(data => {
          this.items = data; this.loading = false});
      }
      },

  async created() {
    this.getDataFromApi()
  },

  methods: {
    async fetchData(page, size) {
      let sortBy = this.sortBy ? this.sortBy : "id"
      const sortDesc = this.sortDesc
      if (sortBy.includes('.')) {
         sortBy = sortBy.split('.')
      }
      let items = sales.results
      const total = sales.results.length
      let sortedItems
      
      let sortA 
      let sortB 
      
      sortedItems = items.sort((a, b) => {
        if (typeof sortBy === "object") {
             sortA = a[sortBy[0]][sortBy[1]]
             sortB = b[sortBy[0]][sortBy[1]]
           }
           else {
               sortA = a[sortBy]
               sortB = b[sortBy]
           }
         
         if (sortDesc) {
           if (sortA < sortB) return 1
           if (sortA > sortB) return -1
           return 0
          } else {
            if (sortA < sortB) return -1
            if (sortA > sortB) return 1
            return 0
          }
        })


      const start = (page-1) * size
      const end = start + size
      await this.delay(3000)
      return await sortedItems.slice(start, end)
    },

    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },

    getDataFromApi() {
      this.items = []
      this.loading = true
      this.fetchData(this.currentPage, this.pageSize).then(data => {
        this.items = data
        this.loading = false
      })
    },

    refresh() {
      if (this.searchQuery.length > 2) {
        this.performAPISearch()
      }
      else {
        this.searchQuery = ''
        this.getDataFromApi() 
      }
    },

    performAPISearch() {
      this.searchItems = [] 
      this.loading = true

      //// CLEAR SEARCH ////
      if (this.activeFilter == null || this.activeFilter == undefined) {
        this.fetchData(1, 999)
        .then(data => data.filter((item) => this.getObjectProps(item)
        .toLowerCase()
        .includes(this.searchQuery.toLowerCase()))).then(result => {
          this.totalSearchItems = result.length; 
          const start = (this.currentPage-1) * this.pageSize
          const end = start + this.pageSize
          this.searchItems = result.slice(start,end)
          this.loading = false})
      }
      //// FILTERED SEARCH ////
      else {        
        const thisFilter = this.activeFilter.includes('.') ? this.activeFilter.split('.') : this.activeFilter
    
        this.fetchData(1, 999)
        .then(data => data.filter((item) => 
        (typeof thisFilter == "object" ? item[thisFilter[0]][thisFilter[1]] : item[thisFilter])
        .toLowerCase()
        .includes(this.searchQuery.toLowerCase())))
        .then(result => {
          this.totalSearchItems = result.length; 
          const start = (this.currentPage-1) * this.pageSize
          const end = start + this.pageSize
          this.searchItems = result.slice(start,end)
          this.loading = false})
      }
    },

  updateCurrentPage(number) {
      this.currentPage = number
      if (this.searchQuery.length > 2) {
        this.performAPISearch()
      }
      else {
        this.getDataFromApi()
      }
    },
    updatePageSize(rows) {
      this.pageSize = rows
      if (this.searchQuery.length > 2) {
        this.performAPISearch()
      }
      else {  
        this.getDataFromApi()
      }
    },
    getObjectProps(obj)  {
          let allProps = ""
          for (const prop in obj) {
            if (typeof obj[prop] === 'object') {
              allProps = allProps.concat(this.getObjectProps(obj[prop]))
            }
            else {
              allProps = allProps.concat(" ", obj[prop])
            }
          }
          return allProps
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