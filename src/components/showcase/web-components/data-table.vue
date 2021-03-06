<template>
  <div>
    <div class="layout-padding">
      <div class="list bordered" style="margin-bottom: 25px">
        <q-collapsible icon="settings" label="Configure">
          <div class="floating-label">
            <input v-model="config.title" required>
            <label>Data Table Title</label>
          </div>

          <p class="caption">Features <small>(* filter can be removed in columns definition)</small></p>
          <div class="column group gt-sm-row">
            <label>
              <q-checkbox v-model="config.refresh" />
              Refresh
            </label>
            <label>
              <q-checkbox v-model="config.columnPicker" />
              Column Picker
            </label>
            <label>
              <q-checkbox v-model="pagination" />
              Pagination
            </label>
            <label>
              <q-checkbox v-model="config.responsive" />
              Responsive
            </label>
          </div>

          <p class="caption">Selection and Sticky Columns</p>
          <div class="column gt-sm-row group">
            <q-select
              v-model="config.selection"
              type="radio"
              label="Selection"
              :options="[
                {label: 'None', value: false},
                {label: 'Single', value: 'single'},
                {label: 'Multiple', value: 'multiple'}
              ]"
            />

            <q-select
              v-model="config.leftStickyColumns"
              type="radio"
              label="Left Sticky Columns *"
              :options="[
                {label: 'None', value: 0},
                {label: '1', value: 1},
                {label: '2', value: 2}
              ]"
            />

            <q-select
              v-model="config.rightStickyColumns"
              type="radio"
              label="Right Sticky Columns *"
              :options="[
                {label: 'None', value: 0},
                {label: '1', value: 1},
                {label: '2', value: 2}
              ]"
            />
          </div>

          <p class="caption">Row Height * ({{rowHeight}}px)</p>
          <q-range v-model="rowHeight" :min="50" :max="200" labelAlways />

          <p class="caption">
            Table body *
            <span :style="{fontStyle: bodyHeightProp === 'auto' ? 'italic' : ''}">({{bodyHeight}}px)</span>
          </p>
          <div class="row items-center">
            <q-select
              type="radio"
              v-model="bodyHeightProp"
              :options="[
                {label: 'Auto', value: 'auto'},
                {label: 'Height', value: 'height'},
                {label: 'Min Height', value: 'minHeight'},
                {label: 'Max Height', value: 'maxHeight'}
              ]"
            />
            <q-range v-model="bodyHeight" :min="100" :max="700" labelAlways :disable="bodyHeightProp === 'auto'" />
          </div>

          <div class="quote light-paragraph" style="margin-top: 35px">* - applies to wide screens only</div>
        </q-collapsible>
      </div>

      <p class="caption desktop-only">Resize browser window to experiment with different screen sizes and see Sticky Columns in action.</p>
      <q-data-table
        :data="table"
        :config="config"
        :columns="columns"
        @refresh="refresh"
      >
        <template slot="col-source" scope="cell">
          <span v-if="cell.data === 'Audit'" class="label text-white bg-primary">
            Audit
            <q-tooltip>Some tooltip</q-tooltip>
          </span>
          <span v-else class="label text-white bg-negative">{{cell.data}}</span>
        </template>

        <template slot="selection" scope="props">
          <button class="primary clear" @click="changeMessage(props)">
            <i>edit</i>
          </button>
          <button class="primary clear" @click="deleteRow(props)">
            <i>delete</i>
          </button>
        </template>
      </q-data-table>
    </div>
  </div>
</template>

<script>
import { Platform, Utils, Toast } from 'quasar'
import table from 'data/table.json'

export default {
  methods: {
    changeMessage (props) {
      props.rows.forEach(row => {
        row.data.message = 'Quasar Framework rocks!'
      })
      Toast.create('Changed "Message" field for selected row(s).')
    },
    deleteRow (props) {
      props.rows.forEach(row => {
        this.table.splice(row.index, 1)
      })
    },
    refresh (done) {
      this.timeout = setTimeout(() => {
        done()
      }, 5000)
    }
  },
  beforeDestroy () {
    if (this.timeout) {
      clearTimeout(this.timeout)
    }
  },
  data () {
    return {
      table,
      config: {
        title: 'Data Table',
        refresh: true,
        columnPicker: true,
        leftStickyColumns: 1,
        rightStickyColumns: 2,
        bodyStyle: {
          maxHeight: Platform.is.mobile ? '50vh' : '500px'
        },
        rowHeight: '50px',
        responsive: true,
        pagination: {
          rowsPerPage: 15,
          options: [5, 10, 15, 30, 50, 500]
        },
        selection: 'multiple',
        messages: {
          noData: '<i>warning</i> No data available to show.',
          noDataAfterFiltering: '<i>warning</i> No results. Please refine your search terms.'
        }
      },
      columns: [
        {
          label: 'Date',
          field: 'isodate',
          width: '120px',
          filter: true,
          sort: 'date',
          format (value) {
            return new Date(value).toLocaleString()
          }
        },
        {
          label: 'Service',
          field: 'serviceable',
          format (value) {
            if (value === 'Informational') {
              return '<i class="text-positive">info</i>'
            }
            return value
          },
          width: '80px'
        },
        {
          label: 'Message',
          field: 'message',
          sort: true,
          filter: true,
          width: '500px'
        },
        {
          label: 'Source',
          field: 'source',
          sort: true,
          filter: true,
          width: '120px'
        },
        {
          label: 'Log Number',
          field: 'log_number',
          sort: true,
          width: '100px'
        }
      ],

      pagination: true,
      rowHeight: 50,
      bodyHeightProp: 'maxHeight',
      bodyHeight: 500
    }
  },
  watch: {
    pagination (value) {
      if (!value) {
        this.oldPagination = Utils.clone(this.config.pagination)
        this.config.pagination = false
        return
      }

      this.config.pagination = this.oldPagination
    },
    rowHeight (value) {
      this.config.rowHeight = value + 'px'
    },
    bodyHeight (value) {
      let style = {}
      if (this.bodyHeightProp !== 'auto') {
        style[this.bodyHeightProp] = value + 'px'
      }
      this.config.bodyStyle = style
    },
    bodyHeightProp (value) {
      let style = {}
      if (value !== 'auto') {
        style[value] = this.bodyHeight + 'px'
      }
      this.config.bodyStyle = style
    }
  }
}
</script>
