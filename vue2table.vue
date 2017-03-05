<template>
<div name="vue2table">
  <div class='clearfix'>
    <div class='pull-left'>
      <input type="text" v-model="inputSearch" @keyup="currentPage = 1" />
    </div>
    <div class='pull-right' v-if="dataTable.rows.length > 0">
      <div class='pull-left panel-perpage'>
        <select v-model="displayRowSelected" class="select-perpage border-color-skin">
          <option v-for="displayRow in displayRowsFilter()">{{ displayRow }}</option>
          <option value="999999">all</option>
        </select>
      </div>
      <div class='pull-right panel-paging'>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 5)"><<</div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 1)"><</div>
        <template v-for="page in totalPage">
          <div class='paging border-color-skin' :class="[ page !== currentPage ? 'color-skin' : 'paging-focus bg-color-skin' ]" @click="setCurrentPage(page)" v-show="filterPage(page)">{{ page }}</div>
        </template>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 1)">></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 5)">>></div>
      </div>
    </div>
  </div>

  <div class="vue2table table-main">
    <table>
      <thead>
        <tr>
          <th v-if="indexDisp">#</th>
          <th v-if="checkboxDisp"><input type="checkbox" v-model="selectAll"/></th>
          <th v-if="radioDisp"></th>
          <th v-for="column in columns" :class="column.class" @click="setSortProperty(column.key)">
            {{ column.name }}
            <template v-show="sortKey == column.key">
              <img src="./assets/sort_both.png" v-if="sortNum == 0">
              <img src="./assets/sort_asc.png" v-if="sortNum == 1">
              <img src="./assets/sort_desc.png" v-if="sortNum == 2">
            </template>
          </th>
        </tr>
      </thead>
      <tbody v-if="dataTable.rows.length > 0">
        <tr v-for="(dataRow, index) in dataTable.rows" v-if="index + 1 >= startRow && index + 1 <= endRow">
          <td v-if="indexDisp" class="align-center">{{ index + 1 }}</td>
          <td v-if="checkboxDisp" class="align-center"><input type="checkbox" :value="dataRow" v-model="checkedDataRows" @click="setCheckboxSelectedRows(checkedDataRows)" /></td>
          <td v-if="radioDisp" class="align-center"><input type="radio" :value="dataRow" v-model="radioDataRow" @click="setRadioSelectedRow(radioDataRow)" /></td>
          <td v-for="column in columns" :class="column.class" :contenteditable="contentEditable">{{ dataRow[column.key] }}</td>
        </tr>
      </tbody>
    </table>
  </div>

  <div class='clearfix' v-if="dataTable.rows.length > 0">
    <div class='pull-left'>Show {{ startRow }} to {{ endRow }} of {{ dataTable.total }}</div>
    <div class='pull-right'>
      <div class='pull-left panel-perpage'>
        <select v-model="displayRowSelected" class="select-perpage border-color-skin">
          <option v-for="displayRow in displayRowsFilter()">{{ displayRow }}</option>
          <option value="999999">all</option>
        </select>
      </div>
      <div class='pull-right panel-paging'>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 5)"><<</div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 1)"><</div>
        <template v-for="page in totalPage">
          <div class='paging border-color-skin' :class="[ page !== currentPage ? 'color-skin' : 'paging-focus bg-color-skin' ]" @click="setCurrentPage(page)" v-show="filterPage(page)">{{ page }}</div>
        </template>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 1)">></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 5)">>></div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import Vue from 'vue'
import Vuex, { mapMutations } from 'vuex'
import _ from 'lodash'

const types = {
  SET_CHECKBOX_SELECTED_ROWS: 'SET_CHECKBOX_SELECTED_ROWS',
  SET_RADIO_SELECTED_ROW: 'SET_RADIO_SELECTED_ROW'
}
Vue.use(Vuex)

const store = new Vuex.Store({
  modules: {
    state: {
      checkboxSelectedRows: [{}],
      radioSelectedRow: {}
    },
    getters: {
      GET_CHECKBOX_SELECTED_ROWS (state) {
        return state.checkboxSelectedRows
      },
      GET_RADIO_SELECTED_ROW (state) {
        return state.radioSelectedRow
      }
    },
    mutations: {
      [types.SET_CHECKBOX_SELECTED_ROWS] (state, checkboxSelectedRows) {
        state.checkboxSelectedRows = checkboxSelectedRows
      },
      [types.SET_RADIO_SELECTED_ROW] (state, radioSelectedRow) {
        state.radioSelectedRow = radioSelectedRow
      }
    }
  }
})

export default {
  name: 'vue2table',
  data () {
    return {
      sortNum: 0,
      sortType: 'sort_both',
      sortKey: '',
      inputSearch: '',
      radioDataRow: {},
      checkedDataRows: [],
      currentPage: this.page,
      displayRowSelected: this.limit,
      displayRows: [ 10, 25, 50, 100, 250, 500, 1000 ]
    }
  },
  props: {
    columns: {
      type: Array,
      required: true,
      default () {
        return {
          key: '',
          name: ''
        }
      }
    },
    data: {
      types: Object,
      required: true,
      default: function () {
        return {
          total: 0,
          rows: []
        }
      }
    },
    page: {
      type: Number,
      default: 1,
      validator: function (value) {
        return value > 0
      }
    },
    limit: {
      type: Number,
      default: 10,
      validator: function (value) {
        return value > 0
      }
    },
    indexDisp: {
      type: Boolean,
      default: false
    },
    checkboxDisp: {
      type: Boolean,
      default: false
    },
    radioDisp: {
      type: Boolean,
      default: false
    },
    customDisp: {
      type: Boolean,
      default: false
    },
    customEvent: {
      type: Function
    },
    customArguments: {
      type: String
    },
    contentEditable: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    displayRowSelected () {
      if (this.dataTable.total / this.displayRowSelected < this.currentPage) {
        this.currentPage = 1
      }
    }
  },
  computed: {
    selectAll: {
      get: function () {
        let dataDisplayRows = (typeof this.dataSort === 'undefined') ? this.data.rows : this.dataSort
        return dataDisplayRows ? this.checkedDataRows.length === dataDisplayRows.length : false
      },
      set: function (value) {
        let dataDisplayRows = (typeof this.dataSort === 'undefined') ? this.data.rows : this.dataSort
        let checkedDataRows = []
        if (value) {
          dataDisplayRows.forEach(function (dataRow) {
            checkedDataRows.push(dataRow)
          })
        }
        this.checkedDataRows = checkedDataRows
        this.setCheckboxSelectedRows(checkedDataRows)
      }
    },
    displayRowSelectedComputed () {
      let displayRowSelected = (parseInt(this.displayRowSelected) <= this.dataTable.total) ? parseInt(this.displayRowSelected) : this.dataTable.total
      return displayRowSelected
    },
    startRow () {
      return (((this.currentPage - 1) * this.displayRowSelectedComputed) + 1)
    },
    endRow () {
      let endRow = (((this.currentPage - 1) * this.displayRowSelectedComputed) + this.displayRowSelectedComputed)
      return (endRow <= this.dataTable.total) ? endRow : this.dataTable.total
    },
    totalPage () {
      return Math.ceil(this.dataTable.total / this.displayRowSelectedComputed)
    },
    dataTable () {
      let dataRows = []
      let dataDisplayRows = (typeof this.dataSort === 'undefined') ? this.data.rows : this.dataSort
      for (let dataRow of dataDisplayRows) {
        for (let column of this.columns) {
          let value = dataRow[column.key]
          if (typeof value === 'string') {
            if (value.includes(this.inputSearch)) {
              dataRows.push(dataRow)
              break
            }
          }
        }
      }
      return {
        total: dataRows.length,
        rows: dataRows
      }
    },
    dataSort () {
      for (let column of this.columns) {
        if (column.key === this.sortKey) {
          if (this.sortNum === 1) {
            return _.sortBy(this.data.rows, column.key)
          } else if (this.sortNum === 2) {
            return _.sortBy(this.data.rows, column.key).reverse()
          }
        }
      }
    }
  },
  methods: {
    displayRowsFilter () {
      let boolPass = true
      let newDisplayRows = []
      for (var displayRow of this.displayRows) {
        if (displayRow < this.dataTable.total) {
          newDisplayRows.push(displayRow)
        } else {
          if (boolPass) {
            newDisplayRows.push(displayRow)
            boolPass = false
          }
        }
      }
      return newDisplayRows
    },
    filterPage (page) {
      let pageIndex = Math.ceil(this.currentPage / 5)
      return (page >= (pageIndex - 1) * 5 + 1 && page <= pageIndex * 5)
    },
    setSortProperty (sortKey) {
      this.sortKey = sortKey
      this.sortNum++
      this.sortNum = (this.sortNum <= 2) ? this.sortNum : 1
    },
    setCurrentPage (page) {
      this.currentPage = (page <= 0) ? 1 : (page > this.totalPage) ? this.totalPage : page
    },
    ...mapMutations({
      setCheckboxSelectedRows: types.SET_CHECKBOX_SELECTED_ROWS,
      setRadioSelectedRow: types.SET_RADIO_SELECTED_ROW
    })
  }
}
</script>

<style lang="scss">
/*table*/
.vue2table {
    .panel-button-control {
        float: left;
        background-color: #e6e6e6;
        padding: 10px;
        width: 100%;
        @media(max-width: 968) {
            text-align: center;
        }
    }
    .table-paging {
        .paging-left {
            margin-top: 13px;
        }
    }
    #table-paging-bottom{
        margin-bottom: 15px;
        background-color: #e6e6e6;
        padding: 0 10px 10px;
        margin-top: 5px;
    }
    .table-border {
        position: relative;
    }
    table {
        .odd{
            background-color: #FFFFFF;
        }
        .even{
            background-color: #EFF4F8;
        }
        thead {
            .tools {
                opacity: 1;
                margin-top: -15px;
                margin-left: -15px;
            }
            .tool-approve{
                height: 30px;
                vertical-align: middle;
                padding-bottom: 0;
            }
            tr {
                th {
                        text-align: center;
                        vertical-align: middle !important;
                        font-weight: normal;
                        border-bottom: 1px solid #ffffff !important;
                }
            }
        }
        tbody {
            background-color: #ffffff;
            color: #000000;
            .selected {
                background-color: #C2DEED;
                &:hover {
                    background-color: #C2DEED;
                }
            }
            tr {
                td {
                    vertical-align: middle;
                    padding: 8px 10px !important;;
                }
                &:hover {
                    background-color: #f5f5f5;
                }
            }
        }

        td, th {
            border: 1px solid #ffffff;
            padding: 5px 5px;
            overflow: auto;
            max-width: 400px;
        }
    }

    &.table-main {
        table {
            width: 100%;
            thead {
                background-color: #3C8DBC;
                color: #ffffff;
                tr {
                    th {
                        &.col-left {
                            border-left: 1px solid #3C8DBC;
                        }
                        &.col-right {
                            border-right: 1px solid #3C8DBC;
                        }
                    }
                }
            }
            tbody {
                tr {
                    height: 50px;
                }
            }
            &.with-filter {
                margin-top: 45px;
            }
        }
    }

    &.table-lov {
        tbody {
            tr {
                height: 35px !important;
            }
        }
    }

    &.table-detail {
        table {
            width: 100%;
            thead {
                background-color: #1A96D6;
                color: #ffffff;
                tr {
                    th {
                        border-top: 1px solid #cccccc;
                        &.col-left {
                            border-left: 1px solid #cccccc;
                        }
                        &.col-right {
                            border-right: 1px solid #cccccc;
                        }
                    }
                }
            }
            &.with-filter {
                margin-top: 45px;
            }
        }
    }

    .tools{
        position: absolute;
        background-color: rgba(121, 121, 121, 0.67);
        color: #e4e4e4;
        opacity: 1;
        line-height: 1;
        margin-left: 0;
        -moz-border-radius: 5px;
        -webkit-border-radius: 5px;
        -khtml-border-radius: 5px;
        border-radius: 4px;
        margin-top: -20px;
        &:hover{
            opacity: 1 !important;
        }
    }
    thead .tools{
        -moz-border-radius: 4px !important;
        -webkit-border-radius: 4px !important;
        -khtml-border-radius: 4px !important;
        border-radius: 4px !important;
    }
    .tool-approve{
        display: table-cell;
        vertical-align: bottom;
        height: 40px;
        width: 85px;
        font-size: 10px;
        cursor: pointer;
        padding-bottom: 3px;
        padding-left: 5px;
        padding-right: 5px;
        .flg-name{
            text-transform: uppercase;
        }
    }
}

.table-header{
    overflow: hidden;
    margin-right: 12px;
}
.table-overflow{
    overflow: auto;
}
.table-overflow-x{
    overflow-x: auto;
}
.table-detail-overflow{
    overflow-x: auto;
    max-height: 500px;
}
.table-lov-overflow{
    overflow-x: auto;
    // max-height: 250px;
}

.paging{
    border: 1px solid #3C8DBC;
    display: block;
    float: left;
    padding: 2px;
    margin-left: 2px;
    min-width: 30px;
    text-align: center;
    color: #3C8DBC;
    margin-top: 10px;
    background-color: #FFF;
}
.paging-focus{
    background-color: #3C8DBC;
    color: #fff;
}
.select-perpage{
    /* for perpage selectbox */
    border: 1px solid #3C8DBC;
    padding: 3px;
    height: 28px;
    margin-top: 10px;
}
.perpage-focus{
    color: #006999;
    font-weight: bold;
}
.panel-perpage{
    margin-right: 20px;
}
.panel-paging{
    /*margin-top: 10px;*/
}
.no-link{
    display: none;
    color: #777;
}
.no-data{
    text-align: center;
    padding: 15px;
}
.td-link{
    width: 100%;
    height: 100%;
    // text-align: center;
}
/*END table*/

/* scrollbar style */
.scrollbar-blue::-webkit-scrollbar-track{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    background-color: #F5F5F5;
}
.scrollbar-blue::-webkit-scrollbar{
    width: 12px;
    height: 12px;
    background-color: #F5F5F5;
}
.scrollbar-blue::-webkit-scrollbar-thumb{
    background-color: #083E4E;
}

.scrollbar-black::-webkit-scrollbar-track{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    background-color: #F5F5F5;
}
.scrollbar-black::-webkit-scrollbar{
    width: 12px;
    height: 12px;
    background-color: #F5F5F5;
}
.scrollbar-black::-webkit-scrollbar-thumb{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
    background-color: #495461;
}

.table-overflow::-webkit-scrollbar-track,
.table-overflow-x::-webkit-scrollbar-track,
.table-detail-overflow::-webkit-scrollbar-track,
.table-lov-overflow::-webkit-scrollbar-track{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    background-color: #F5F5F5;
}

.table-overflow::-webkit-scrollbar,
.table-overflow-x::-webkit-scrollbar,
.table-detail-overflow::-webkit-scrollbar,
.table-lov-overflow::-webkit-scrollbar{
    width: 12px;
    height: 12px;
    background-color: #F5F5F5;
}

.table-overflow::-webkit-scrollbar-thumb,
.table-overflow-x::-webkit-scrollbar-thumb,
.table-detail-overflow::-webkit-scrollbar-thumb,
.table-lov-overflow::-webkit-scrollbar-thumb{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
    background-color: #B8BBBD;
}

.click-link{
    cursor:pointer;
}

.well-width{
    margin-left: 3px;
    margin-right: 3px;
    border-radius: 0 0 4px 4px;
    padding: 10px;
    margin-bottom: 10px;
}

.pull-right {
  float: right !important;
}
.pull-left {
  float: left !important;
}
</style>