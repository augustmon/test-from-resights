<template lang="pug">
v-container
  v-row 
    v-col(md=3)
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

        :headers="headers"
        :items="searchQuery.length < 3 ? items : filteredEntries"
        
        @new-page-number="(page) => updateCurrentPage(page)"
        @new-page-size="(rows) => updatePageSize(rows)"
        :currentPage.sync="currentPage"
        :pageSize.sync="pageSize"

        :loading="loading"
      )

        //- v-if="items.length"
        //- v-progress-circular(
        //-   v-else
        //-   width="2"
        //-   color="rs__primary"
        //-   indeterminate
        //- ).mx-auto

    //- <v-pagination v-model="currentPage" :length="100" :total-visible="10"> </v-pagination>
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
      searchQuery: ''

    }
  },

  computed: {
    filteredEntries() {
      if (this.searchQuery.length > 2) {

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
        console.log("searchQuery: ", this.searchQuery )
        return this.items.filter((item) => getObjectProps(item).toLowerCase().includes(this.searchQuery))
      }
    },   
  },

  watch: {
    // async pageNumber(number) {
    //   if (this.fetchedEntries < (this.currentPage*this.pageSize) + this.pageSize ) {
    //     let pageLoad = await this.fetchData(Math.ceil(this.fetchedEntries)/this.pageSize, this.pageSize)
    //     this.items = [...this.items, ...pageLoad]
    //     this.fetchedEntries += pageLoad.length
    //   }
    // }
  },

  async created() {
    this.getDataFromApi()
  },

  methods: {
    async fetchData(page, size) {
      const start = (page-1) * size
      const end = start + size
      await this.delay(3000)

      return await sales.results.slice(start, end)
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


    async updateCurrentPage(number) {
      this.currentPage = number
      this.getDataFromApi()
    },
    updatePageSize(rows) {
      this.pageSize = rows
      this.getDataFromApi()
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