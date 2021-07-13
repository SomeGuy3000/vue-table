<template>
  <div class="Label my-2">
    <div v-if="endpointRes && endpointRes != null && endpointError == null">
      <div class="float-right my-2 " v-if="search === true">
        Search:
        <input class="text-gray-600 border border-gray-600 rounded-sm px-2" type="text" v-model="searchForItem" placeholder="Type something here.." @input="searchChanged">
      </div>
      <table class="table-auto border-collapse w-full">
        <thead>
          <tr class="text-gray-600">
            <th v-for="path in splitDisplayOrder" :key="path" v-bind:path="path" @click="sort(path)" >{{capitalizeFirstLetter(path.replace(/\./g,' '))}}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="obj in sortedRows" :key="obj" class="even:bg-gray-100">
            <td class="border border-gray-600 px-2 py-1" v-for="path in splitDisplayOrder" :key="path" >
              <div v-if="path == 'email'">
                <a class="underline" v-bind:href="'mailto:' + obj[path]">
                  {{obj[path]}}
                </a>
              </div>
              <div v-else-if="path == 'website'">
                <a class="underline" target="_blank" v-bind:href="'https://' + obj[path]">
                  {{obj[path]}}
                </a>
              </div>
              <div v-else>
                {{obj[path]}}
              </div>
            </td>
          </tr>
        </tbody>
      </table>
      <div class="mx-2 my-2" v-if="pagination === true && displayedRows.length >= pageSize">
        <button class="float-left border border-gray-600 rounded-sm bg-gray-600 text-white px-3 py-0.5" v-if="currentPage != 1" @click="prevPage">Previous</button> 
        <button class="float-right border border-gray-600 rounded-sm bg-gray-600 text-white px-3 py-0.5" v-if="currentPage * pageSize < displayedRows.length" @click="nextPage">Next</button>
      </div>
    </div>
    <div v-else class="grid justify-items-center">
      <h1 class="text-xl">Data cannot be obtained</h1>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Label',
  props: {
    endpoint: String,
    updateOnEndpointChange: Boolean,
    displayOrder: String,
    search: Boolean,
    sorting: Boolean,
    pagination: Boolean,
  },
  data () {
    return {
      endpointRes: null,
      splitDisplayOrder: this.displayOrder.split(","),
      tabRows: [],
      displayedRows: [],
      currentSort:'name',
      currentSortDir:'asc',
      pageSize:3,
      currentPage:1,
      searchForItem: '',
      endpointError: null,
      endpointRefreshCooldown: 10000
    }
  },
  methods: {
    async getDataFromEndpoint (endpoint) {
      let response = await axios.get(endpoint).catch((err) => {this.endpointError = err})
      if (response.status){
        return response.data
      }

    },
    capitalizeFirstLetter (string) {
      return string.charAt(0).toUpperCase() + string.slice(1)
    },
    getJsonElByPath (obj, path) {  
      if (path && obj) {
        path.forEach(function(key){
          obj = obj[key]
        })
        return obj
      }
    },
    async tabElements (endpointRes, splitDisplayOrder) {
      if (endpointRes && splitDisplayOrder){
        endpointRes.forEach ((obj) => {
          let data = ""
          splitDisplayOrder.forEach ((element) => {
            data+=`"${element}":"${this.getJsonElByPath(obj, element.split('.'))}",`
          })
          data = JSON.parse("{" + data.slice(0, -1) + "}")
          this.tabRows.push(data)
        })
      } else {
        this.endpointRes = await this.getDataFromEndpoint(this.endpoint)
      }
    },
    sort (sortBy) {
      if (this.sorting === true) {
        if(sortBy === this.currentSort) {
          this.currentSortDir = this.currentSortDir==='asc'?'desc':'asc'
        }
        this.currentSort = sortBy
      }
    },
    nextPage () {
      if((this.currentPage*this.pageSize) < this.tabRows.length) this.currentPage++;
    },
    prevPage () {
      if(this.currentPage > 1) this.currentPage--;
    },
    searchChanged () {
      this.currentPage = 1
    }
  },
  async mounted () {
    if (this.endpoint){
      this.endpointRes = await this.getDataFromEndpoint(this.endpoint)
      this.tabElements(this.endpointRes, this.splitDisplayOrder)
      if (this.updateOnEndpointChange === true){
        await setInterval(async () => { 
          let res = await this.getDataFromEndpoint(this.endpoint)
          if (JSON.stringify(res) != JSON.stringify(this.endpointRes) && res){
            this.tabRows = []
            this.endpointRes = res
            this.tabElements(this.endpointRes, this.splitDisplayOrder)
          }
        }, this.endpointRefreshCooldown);
      }
    }
  },
  computed: {
    sortedRows () {
      // eslint-disable-next-line vue/no-side-effects-in-computed-properties
      let sortedRows = this.tabRows.sort((a,b) => {
          let modifier = 1
          if(this.currentSortDir === 'desc') modifier = -1
          if(a[this.currentSort] < b[this.currentSort]) return -1 * modifier
          if(a[this.currentSort] > b[this.currentSort]) return 1 * modifier
          return 0
        })
      if (this.pagination === false) {
        if (this.searchForItem != ""){
          return this.tabRows.filter(item => {
            return item.name.toLowerCase().indexOf(this.searchForItem.toLowerCase()) > -1
          })
        }
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.displayedRows = sortedRows;
        return sortedRows
      }
      else{
        if (this.searchForItem != ""){
          let filteredRows = this.tabRows.filter(item => {
            return item.name.toLowerCase().indexOf(this.searchForItem.toLowerCase()) > -1
          })
          // eslint-disable-next-line vue/no-side-effects-in-computed-properties
          this.displayedRows = filteredRows
          return filteredRows.filter((row, index) => {
            let start = (this.currentPage-1)*this.pageSize
            let end = this.currentPage*this.pageSize
            if(index >= start && index < end) return true
          })
        }
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.displayedRows = sortedRows
        return sortedRows.filter((row, index) => {
          let start = (this.currentPage-1)*this.pageSize
          let end = this.currentPage*this.pageSize
          if(index >= start && index < end) return true
        })
      }
    },
  }
}
</script>