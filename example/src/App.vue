<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>{{ msg }}</h1>
    <vue2table :columns="columns" :data="data" />
  </div>
</template>

<script>
import vue2table from './components/vue2table.vue'
import faker from 'faker'

let mockFields = [{
  key: 'title',
  name: 'Title',
  sortNum: 0
}, {
  key: 'detail',
  name: 'Detail',
  sortNum: 0
}]
let mockRows = []
const mockTotal = 100
for (var i = 1; i <= mockTotal; i++) {
  mockRows.push({
    title: faker.lorem.word(),
    detail: faker.lorem.sentence()
  })
}

export default {
  name: 'app',
  data () {
    return {
      msg: 'Vue Hrmstable',
      columns: mockFields,
      data: {
        total: mockTotal,
        rows: mockRows
      }
    }
  },
  components: {
    vue2table
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
