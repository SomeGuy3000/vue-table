<template>
  <div class="Label my-2">
    <table class="table-auto border-collapse w-full">
      <thead>
        <tr class="text-gray-600">
          <th v-for="path in splitDisplayOrder" :key="path">{{capitalizeFirstLetter(path.replace(/\./g,' '))}}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="obj in endpointRes" :key="obj" class="even:bg-gray-100">
          <td class="border border-gray-600 px-2 py-1" v-for="path in splitDisplayOrder" :key="path" >
            <div v-if="path == 'email'">
              <a class="underline" v-bind:href="'mailto:' + obj[path]">
                {{getJsonElByPath(obj, path.split('.'))}}
              </a>
            </div>
            <div v-else>
              {{getJsonElByPath(obj, path.split('.'))}}
            </div>
          </td>
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
    displayOrder: String,
    search: Boolean,
    sorting: Boolean,
    pagination: Boolean,
  },
  data () {
    return {
      endpointRes: null,
      splitDisplayOrder: this.displayOrder.split(",")
    }
  },
  methods: {
    capitalizeFirstLetter (string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    getJsonElByPath (obj, path) {  
      if (path && obj) {
        path.forEach(function(key){
          obj = obj[key];
        });
        console.log(obj)
        return obj;
      }
    }
  },
  mounted () {
    axios
      .get(this.endpoint)
      .then(response => {this.endpointRes = response.data
      })
  }
}
</script>