<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>{{ msg }}</h1>
    <div v-for="(val, key) in selected">
      {{ key }} : {{ val }}
    </div>
    <!-- 
      Props Detail:
      columns: array, required, default [column]
        column.key: string, required, default ''
        column.name: string, required, default ''
        column.class: string, optional, ex. 'align-center',
        column.style: string, optional, ex. 'min-width: 100px;',
        column.inlineType: string, optional ('text', 'textarea', 'select'),
        column.headerRow: number, optional, default 1
                          description: 1, 2, 3... for display at first/second/third of header row respectively
        column.colspan: number, optional, default 1
                        description: 1, 2, 3... for display total one, two, three column at column.headerRow
        column.rowspan: number, optional,
                        description: 1, 2, 3... for display total one, two, three row count from column.headerRow
      maxRowspan: number, optional, default 1
      data: object, required, default { total: 0,rows: []}
      page: number, optional, default 1
      limit: number, optional (10, 25, 50, 100, 250, 500, 1000), default 10
      indexDisp: boolean, optional, default false, 
                 description: true/false for display/hide sequence column
      multiSelect: boolean, optional, 
                   description: true for display checkbox column, false for display radio column
        v-model: use with multiSelect property for received selected row from radio as {} or checkbox as [{}]
    -->
    <vue2table :columns="columns" 
                :data="data" 
                :page="1" 
                :limit="10" 
                :indexDisp="true"
                :multiSelect="false"
                v-model="selected"
                :maxRowspan="maxRowspan" />
  </div>
</template>

<script>
import vue2table from './components/vue2table.vue'
import faker from 'faker'
import _ from 'lodash'

let mockStatusDropdown = []
const totalDropdown = 5 + Math.floor(Math.random() * 6)
for (var i = 1; i < totalDropdown; i++) {
  mockStatusDropdown.push({
    code: _.padStart(Math.floor(Math.random() * 1000), 4, '0'),
    descCode: faker.lorem.word()
  })
}
let mockDropdowns = {
  status: mockStatusDropdown
}
let mockMaxRowspan = 2
let mockColumns = [{
  key: 'firstname',
  name: 'First Name',
  style: 'min-width: 100px;',
  inlineType: 'text'
}, {
  key: 'lastname',
  name: 'Last Name',
  style: 'min-width: 100px;',
  inlineType: 'text'
}, {
  key: 'jobDescriptor',
  name: 'Job',
  class: 'align-center',
  style: 'min-width: 120px;'
}, {
  key: 'date',
  name: 'Date',
  class: 'align-center',
  style: 'min-width: 240px;',
  headerRow: 1,
  colspan: 2,
  rowspan: 1
}, {
  key: 'birthday',
  name: 'Birth',
  class: 'align-center',
  style: 'min-width: 120px;',
  headerRow: 2,
  colspan: 1,
  rowspan: 1
}, {
  key: 'workdate',
  name: 'Work',
  class: 'align-center',
  style: 'min-width: 120px;',
  headerRow: 2,
  colspan: 1,
  rowspan: 1
}, {
  key: 'time',
  name: 'Time',
  class: 'align-center',
  style: 'min-width: 160px;',
  headerRow: 1,
  colspan: 2,
  rowspan: 1
}, {
  key: 'timin',
  name: 'Time in',
  class: 'align-center',
  style: 'min-width: 80px;',
  headerRow: 2,
  colspan: 1,
  rowspan: 1
}, {
  key: 'timout',
  name: 'Time out',
  class: 'align-center',
  style: 'min-width: 80px;',
  headerRow: 2,
  colspan: 1,
  rowspan: 1
}, {
  key: 'status',
  name: 'Status',
  style: 'min-width: 50px;',
  inlineType: 'select',
  inlineDropdown: mockDropdowns.status
}, {
  key: 'remark',
  name: 'Remark',
  style: 'min-width: 200px;',
  inlineType: 'textarea'
}]
let mockRows = []
const mockTotal = 100
for (var j = 1; j <= mockTotal; j++) {
  mockRows.push({
    id: j,
    firstname: faker.name.firstName(),
    lastname: faker.name.lastName(),
    jobDescriptor: faker.name.jobDescriptor(),
    birthday: faker.date.past().toString().substr(4, 12),
    workdate: faker.date.future().toString().substr(4, 12),
    timin: faker.date.past().toString().substr(16, 5),
    timout: faker.date.past().toString().substr(16, 5),
    status: _.padStart(Math.floor(Math.random() * 1000), 4, '0'),
    remark: faker.lorem.sentence()
  })
}

export default {
  name: 'test1',
  data () {
    return {
      msg: 'Vue2 Table',
      columns: mockColumns,
      maxRowspan: mockMaxRowspan,
      data: {
        total: mockTotal,
        rows: mockRows
      },
      selected: {}
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