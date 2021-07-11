<template>
  <div class="Label my-2">
    <table class="table-auto border-collapse w-full">
      <thead>
        <tr class="text-gray-600">
          <th>Name</th>
          <th>Email</th>
          <th>Company name</th>
          <th>City</th>
          <th>Website</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="obj in endpointRes" :key="obj" class="even:bg-gray-100">
            <td class="border border-gray-600 px-2 py-1">{{obj.name}}</td>
            <td class="border border-gray-600 px-2 py-1">{{obj.email}}</td>
            <td class="border border-gray-600 px-2 py-1">{{obj.company.name}}</td>
            <td class="border border-gray-600 px-2 py-1">{{obj.address.city}}</td>
            <td class="border border-gray-600 px-2 py-1">{{obj.website}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Label',
  props: {
    endpoint: String,
    search: Boolean,
    sorting: Boolean,
    pagination: Boolean,
  },
  data () {
    return {
      endpointRes: null
    }
  },  
  mounted () {
    axios
      .get(this.endpoint)
      .then(response => {this.endpointRes = response.data
      console.log(this.endpointRes)
      })
  }
}
</script>