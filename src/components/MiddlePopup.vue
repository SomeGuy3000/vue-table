<template>
  <div class="MiddlePopup m-6">
    <div class="max-w-3xl mx-auto bg-white rounded-xl shadow-md overflow-hidden md:max-w-3xl p-4">
        <Table endpoint="https://jsonplaceholder.typicode.com/users" v-bind:updateOnEndpointChange="true" displayOrder="name,email,company.name,address.city,website" v-bind:search="true" v-bind:sorting="true" v-bind:pagination="true"/>
    </div>
  </div>
</template>

<script>
import Table from './Table.vue'

export default {
  name: 'MiddlePopup',
    components: {
    Table
  },
  props: {}
}
</script>