<template>
<div name="vue2table" class="wrapper-vue2table">
  <div class='clearfix'>
    <div class='pull-left paging-left'>
      <input type="text" v-model="inputSearch" @keyup="currentPage = 1" />
    </div>
    <div class='pull-right paging-right' v-if="dataTable.rows.length > 0">
      <div class='pull-left panel-perpage'>
        <select v-model="displayRowSelected" class="select-perpage border-color-skin">
          <option v-for="displayRow in displayRowsFilter()">{{ displayRow }}</option>
          <option value="999999">all</option>
        </select>
      </div>
      <div class='pull-right panel-paging'>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 5)"><i class='fa fa-angle-double-left'></i></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 1)"><i class='fa fa-angle-left'></i></div>
        <template v-for="page in totalPage">
          <div class='paging border-color-skin' :class="[ page !== currentPage ? 'color-skin' : 'paging-focus bg-color-skin' ]" @click="setCurrentPage(page)" v-show="filterPage(page)">{{ page }}</div>
        </template>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 1)"><i class='fa fa-angle-right'></i></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 5)"><i class='fa fa-angle-double-right'></i></div>
      </div>
    </div>
  </div>

  <div class="vue2table table-main">
    <table>
      <thead>
        <tr v-for="headerRow in maxRowspan" v-if="filterHeaderColumns(columns, headerRow).length > 0">
          <template v-if="headerRow == 1">
            <th v-if="indexDisp" :rowspan="maxRowspan" class="fix-position"><i class="fa fa-hashtag"></i></th>
            <template v-if="typeof multiSelect == 'boolean'">
              <th v-if="multiSelect" :rowspan="maxRowspan" class="fix-position"><input type="checkbox" v-model="selectAll"/></th>
              <th v-if="!multiSelect" :rowspan="maxRowspan" class="fix-position"><i class="fa fa-circle-o"></i></th>
            </template>
          </template>
          <template v-for="column in filterHeaderColumns(columns, headerRow)">
            <template v-if="column.colspan == 1">
              <th :colspan="column.colspan" :rowspan="column.rowspan" :class="column.class" :style="column.style" @click="setSortProperty(column.key)">
                {{ column.name }}
                <template v-show="sortKey == column.key">
                  <img src="./assets/sort_asc.png" v-if="sortNum == 1">
                  <img src="./assets/sort_desc.png" v-if="sortNum == 2">
                </template>
              </th>
            </template>
            <template v-else>
              <th :colspan="column.colspan" :rowspan="column.rowspan" :class="column.class" :style="column.style">
                {{ column.name }}
              </th>
            </template>
          </template>
        </tr>
      </thead>
      <tbody v-if="dataTable.rows.length > 0">
        <tr v-for="(dataRow, index) in dataTable.rows" v-if="index + 1 >= startRow && index + 1 <= endRow">
          <td v-if="indexDisp" class="align-center">{{ index + 1 }}</td>
          <template v-if="typeof multiSelect == 'boolean'">
            <td v-if="multiSelect" class="align-center"><input type="checkbox" :value="dataRow" v-model="selectedRows" /></td>
            <td v-if="!multiSelect" class="align-center"><input type="radio" :value="dataRow" v-model="selectedRows" /></td>
          </template>
          <template v-for="column in filterBodyColumns(columns)">
            <td :class="column.class">
              <template v-if="typeof column.inlineType == 'undefined'">
                {{ dataRow[column.key] }}
              </template>
              <template v-else>
                <template v-if="column.inlineType == 'text'">
                  <input type="text" v-model="dataRow[column.key]" />
                </template>
                <template v-else-if="column.inlineType == 'select'">
                  <select v-model="dataRow[column.key]">
                    <option v-for="obj in column.inlineDropdown" :value="obj.code">{{ obj.descCode }}</option>
                  </select>
                </template>
                <template v-else-if="column.inlineType == 'textarea'">
                  <textarea v-model="dataRow[column.key]" />
                </template>
              </template>
            </td>
          </template>
        </tr>
      </tbody>
    </table>
  </div>

  <div class='clearfix' v-if="dataTable.rows.length > 0">
    <div class='pull-left paging-left'>Show {{ startRow }} to {{ endRow }} of {{ dataTable.total }}</div>
    <div class='pull-right paging-right'>
      <div class='pull-left panel-perpage'>
        <select v-model="displayRowSelected" class="select-perpage border-color-skin">
          <option v-for="displayRow in displayRowsFilter()">{{ displayRow }}</option>
          <option value="999999">all</option>
        </select>
      </div>
      <div class='pull-right panel-paging'>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 5)"><i class='fa fa-angle-double-left'></i></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage - 1)"><i class='fa fa-angle-left'></i></div>
        <template v-for="page in totalPage">
          <div class='paging border-color-skin' :class="[ page !== currentPage ? 'color-skin' : 'paging-focus bg-color-skin' ]" @click="setCurrentPage(page)" v-show="filterPage(page)">{{ page }}</div>
        </template>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 1)"><i class='fa fa-angle-right'></i></div>
        <div class='paging color-skin border-color-skin' @click="setCurrentPage(currentPage + 5)"><i class='fa fa-angle-double-right'></i></div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'vue2table',
  data () {
    return {
      sortNum: 0,
      sortType: 'sort_both',
      sortKey: '',
      inputSearch: '',
      selectedRows: this.value,
      currentPage: this.page,
      displayRowSelected: this.limit,
      displayRows: [ 10, 25, 50, 100, 250, 500, 1000 ]
    }
  },
  props: {
    value: {},
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
    maxRowspan: {
      type: Number,
      default: 1
    },
    indexDisp: {
      type: Boolean,
      default: false
    },
    multiSelect: {}
  },
  watch: {
    selectedRows () {
      this.$emit('input', this.selectedRows)
    },
    displayRowSelected () {
      if (this.dataTable.total / this.displayRowSelected < this.currentPage) {
        this.currentPage = 1
      }
    },
    sortNum () {
      if (this.sortNum === 0) {
        this.sortType = 'sort_both'
      } else if (this.sortNum === 1) {
        this.sortType = 'sort_asc'
      } else {
        this.sortType = 'sort_desc'
      }
    }
  },
  computed: {
    selectAll: {
      get: function () {
        let dataDisplayRows = (typeof this.dataSort === 'undefined') ? this.data.rows : this.dataSort
        return dataDisplayRows ? this.selectedRows.length === dataDisplayRows.length : false
      },
      set: function (value) {
        let dataDisplayRows = (typeof this.dataSort === 'undefined') ? this.data.rows : this.dataSort
        let selectedRows = []
        if (value) {
          dataDisplayRows.forEach(function (dataRow) {
            selectedRows.push(dataRow)
          })
        }
        this.selectedRows = selectedRows
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
      let filterBodyColumns = this.filterBodyColumns(this.columns)
      for (let dataRow of dataDisplayRows) {
        for (let column of filterBodyColumns) {
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
      let filterBodyColumns = this.filterBodyColumns(this.columns)
      for (let column of filterBodyColumns) {
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
    filterHeaderColumns (columns, headerRow) {
      let maxRowspan = this.maxRowspan
      return _.filter(columns, function (column) {
        column.headerRow = (typeof column.headerRow === 'undefined') ? 1 : parseInt(column.headerRow)
        column.colspan = (typeof column.colspan === 'undefined') ? 1 : parseInt(column.colspan)
        column.rowspan = (typeof column.rowspan === 'undefined') ? maxRowspan : parseInt(column.rowspan)
        return column.headerRow === headerRow
      })
    },
    filterBodyColumns (columns) {
      return _.filter(columns, function (column) {
        column.colspan = (typeof column.colspan === 'undefined') ? 1 : parseInt(column.colspan)
        return column.colspan === 1
      })
    }
  }
}
</script>

<style lang="scss">
/*table*/
.wrapper-vue2table{
    margin:20px 0;

    .vue2table {
        margin:10px 0;
        /*overflow:auto;*/

        .panel-button-control {
            float: left;
            background-color: #e6e6e6;
            padding: 10px;
            width: 100%;
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
                            padding:10px 5px;
                            font-size:16px;
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
        /* for perpage text */
        /*margin-left: 2px;
        cursor: default;*/
    }
    .perpage-focus{
        color: #006999;
        font-weight: bold;
    }
    .panel-perpage{
        margin-right: 20px;
        /* margin-top: 13px;*/   /* for perpage text */
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