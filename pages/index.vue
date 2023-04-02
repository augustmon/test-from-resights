<template lang="pug">
v-container
    input( 
      type='text' 
      v-model="searchQuery" 
      placeholder='Search for a record...')

    v-row
      v-col(cols)
        DataTable(
          v-if="items.length"
          :headers="headers"
          :items="searchQuery.length < 3 ? items : filteredEntries"
        )
        v-progress-circular(
          v-else
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto
        //- <p> {{ searchQuery }} </p> 
        //- <ol> 
        //- <li v-if="searchQuery.length > 2" v-for="entry in filteredEntries" :key="entry.id"> {{entry.user.first_name + " " + entry.user.last_name}}</li>
        //- </ol>

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
      searchQuery: ''
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

  async created() {
    this.items = await this.fetchData(0, 50)
  },

  methods: {
    async fetchData(page, size) {
      const start = page * size
      await this.delay(3000)
      return await sales.results.slice(start, start + size)
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
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