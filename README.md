Vue2table
=========

Generated table for vuex feature

## Required

   - font-awesome
   - lodash

## Installation

  `npm i vue2table -S`

## Examples

  - See [examples](https://github.com/Nutchawat/vue2table/tree/master/example) folder

## Roadmaps
  
  - Render Client Columns with colspan, rowspan designable (complete)
  - Render Client Data (complete)
  - Pagination (complete)
  - Search Data (complete)
  - Select Data row with vue (complete)
  - Sort Data string (complete)
  - Inline Edit text, textarea, select (complete)

  - Select Data row with vuex (coming soon...)
  - Sort Data number, date, boolean (coming soon...)
  - Inline Edit date, time (coming soon...)
  - Filter Column (coming soon...)
  - Drag & drop columns (coming soon...)
  - Custom Dom & Event (coming soon...)

## Font Awesome Registeration

```js
// App.js
import 'font-awesome/css/font-awesome.min.css'
```


## Component Registeration

  - Register vue2table with global components

```js
...
import Vue from 'vue'
import vue2table from 'vue2table'
...

...
Vue.component('vue2table', Vue2table)`
...
```

  - or Register vue2table with local component

```js
<script>
import vue2table from 'vue2table'
    
export default {
  ...
  components: {
    'any-component-name': vue2table
  }
}
</script>
```

## Usage
 
```js 
  <vue2table :columns="columns" :data="data" />
```

## Props

  - columns: array, required, default [column]
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
  - maxRowspan: number, optional, default 1
  - data: object, required, default { total: 0,rows: []}
  - page: number, optional, default 1
  - limit: number, optional (10, 25, 50, 100, 250, 500, 1000), default 10
  - indexDisp: boolean, optional, default false, 
               description: true/false for display/hide sequence column
  - multiSelect: boolean, optional, 
                 description: true for display checkbox column, false for display radio column
      v-model: use with multiSelect property for received selected row from radio as {} or checkbox as [{}]

## Contributing

  - See [Contributors](https://github.com/Nutchawat/vue2table/graphs/contributors)

## License

  - [MIT License](https://github.com/Nutchawat/vue2table/blob/master/LICENSE.md)
