<template>
  <div class="app-container">
    <el-row>
      <data-source-select
        v-if="getLengthGtZore(selectServers)"
        :items="selectServers"
        @getValue="handlerDataSource"
        :placeholder="'ClickHouse Server'">
      </data-source-select>
      <el-button v-if="getLengthGtZore(selectServers)"
        type="primary"
        icon="el-icon-edit"
        size="mini"
        :loading="executeLoading"
        @click="handlerExecute()">
        {{ this.$t('common.execute') }}
      </el-button>
      <el-tooltip class="item" effect="dark" content="Actual execution process will not be cancelled!" placement="bottom">
        <el-button
          type="danger"
          size="mini" 
          :disabled="disabled.cancel" 
          @click="handlerCancel()"> {{ this.$t('common.cancel') }}
        </el-button>
      </el-tooltip>
      <el-tooltip class="item" effect="dark" content="Only 100 records are recorded and displayed!" placement="bottom">
        <el-button
          type="primary"
          size="mini" 
          @click="disabled.history = true">
          <i class="fa fa-history"></i>
        </el-button>
      </el-tooltip>
      <el-tooltip class="item" effect="dark" content="Add New DataSource" placement="bottom">
        <el-button type="primary" size="mini" style="float: right;" :disabled="disabled.quickQuery" @click="disabled.newDataSource = true">
          <i class="fa fa-plus-circle"></i>
        </el-button>
      </el-tooltip>
      <el-button
        v-if="getLengthGtZore(selectServers)"
        type="success"
        icon="el-icon-more"
        size="mini"
        style="float: right;"
        :disabled="disabled.quickQuery" 
        @click="loading.quickQuery = true">
        {{ this.$t('common.quick_query') }}
      </el-button>
    </el-row>
    <el-row v-loading="executeLoading">
      <textarea ref='mycode' class='codesql' v-model='code'></textarea>
    </el-row>
    <el-row v-if="data.statistics" v-loading="executeLoading">
      <el-tag size="mini">
        <i class="fa fa-clock-o"></i> Elapsed Time {{ data.statistics.elapsed }} sec
      </el-tag>
      <el-tag type="success" size="mini">
        <i class="fa fa-grip-lines"></i> Total Rows {{ data.rows }} rows
      </el-tag>
      <el-tag type="success" size="mini">
        <i class="fa fa-adjust"></i> Total Read Rows {{ data.statistics.rows_read }} row
      </el-tag>
      <el-tag type="success" size="mini">
        <i class="fa fa-perbyte"></i> Bytes Read {{ data.statistics.bytes_read }} bytes
      </el-tag>
    </el-row>
    <el-row>
      <table-detail v-if="data.headers" :showIndex="true" :columns="data.columns" :headers="data.headers" :loading="executeLoading"></table-detail>
    </el-row>
    <query-quick :loading="loading.quickQuery" :width="'70%'" @close="loading.quickQuery = false" @getQuickSql="handlerGetQuickSql"></query-quick>
    <query-history :loading="disabled.history" :width="'80%'" @close="disabled.history = false"></query-history>
    <data-source :title="'Add New DataSource'"
      :loading="disabled.newDataSource"
      @close="disabled.newDataSource = false"
      @refresh="_initializeServer"></data-source>
  </div>
</template>

<script>
import TableDetail from '@/components/Table'
import QueryQuick from '@/views/components/query/QueryQuick'
import QueryHistory from '@/views/components/query/QueryHistory'
import DataSource from '@/views/components/data/datasource/DataSource'
import DataSourceSelect from '@/views/components/data/datasource/DataSourceSelect'
import { getQuery } from '@/services/Query'
import { getDataSources } from '@/services/DataSource'

import 'codemirror/theme/ambiance.css'
import 'codemirror/lib/codemirror.css'
import 'codemirror/addon/hint/show-hint.css'

const CodeMirror = require('codemirror/lib/codemirror')
require('codemirror/addon/edit/matchbrackets')
require('codemirror/addon/selection/active-line')
require('codemirror/mode/sql/sql')
require('codemirror/addon/hint/show-hint')
require('codemirror/addon/hint/sql-hint')

export default {
  name: 'codeMirror',
  components: {
    TableDetail,
    QueryQuick,
    QueryHistory,
    DataSource,
    DataSourceSelect
  },
  data() {
    return {
      editor: null,
      code: '',
      data: {},
      executeLoading: false,
      inputValue: '',
      selectValue: null,
      selectServers: [],
      loading: {
        cancel: false,
        quickQuery: false
      },
      disabled: {
        cancel: true,
        quickQuery: false,
        history: false,
        newDataSource: false
      }
    }
  },
  mounted() {
    this._initialize()
  },
  methods: {
    _initialize() {
      const mime = 'text/x-mariadb'
      this.editor = CodeMirror.fromTextArea(this.$refs.mycode, {
        mode: mime,
        indentWithTabs: true,
        smartIndent: true,
        lineNumbers: true,
        matchBrackets: true,
        extraKeys: {
          'Ctrl': 'autocompconste'
        }
      })
      this._initializeServer()
    },
    _initializeServer() {
      this.selectServers = getDataSources(null).columns
    },
    async handlerExecute() {
      this.executeLoading = true
      this.disabled.cancel = false
      this.disabled.quickQuery = true
      const response = await getQuery(this.selectValue, this.editor.getValue())
      if (!response.status) {
        this.$notify.error({
          title: 'Error',
          message: response.message
        })
      } else {
        if (response.message) {
          this.$notify({
            title: 'Notification',
            type: 'success',
            message: 'Operation successful!'
          })
        } else {
          this.data = response
        }
      }
      this.executeLoading = false
      this.disabled.quickQuery = false
      this.disabled.cancel = true
    },
    handlerGetQuickSql(value) {
      this.editor.setValue(value)
    },
    handlerCancel() {
      this.disabled.cancel = true
      this.disabled.quickQuery = false
      this.executeLoading = false
    },
    handlerDataSource(value) {
      this.selectValue = value
    }
  }
}
</script>

<style scoped>
  .el-row {
    margin-top: 10px;
  }
</style>
