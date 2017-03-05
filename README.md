Vue2table
=========

Generated table for vuex feature

## Required

   - vuex
   - lodash

## Installation

  `npm i vuex lodash`
  
  `npm i vue2table -S`

## Registeration

  - Register vue2table with global components

    `import Vue from 'vue'
    import vue2table from 'vue2table'

    Vue.component('vue2table', Vue2table)`

  - or Register vue2table with local component

    `import vue2table from 'vue2table'
    
    export default {
      name: 'vue2table',
      components: {
        vue2table
      }
    }`

## Usage
  
  `<vue2table :columns="columns" :data="data" />`

## Tests

  `npm test`

## Contributing

