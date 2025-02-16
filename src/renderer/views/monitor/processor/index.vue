<template>
  <div class="app-container">
    <el-row>
      <el-form :inline="true" :model="form" size="mini">
        <el-form-item :label="this.$t('common.server')">
          <data-source-select
            v-if="getLengthGtZore(selectServers)"
            :items="selectServers"
            @getValue="handlerServer"
            :placeholder="'ClickHouse Server'">
          </data-source-select>
          <span v-if="form.auto">
            <i class="fa fa-spin fa-spinner"></i>
            {{ this.$t('common.refresh') + this.$t('common.time') }} <el-tag size="mini"> {{ this.refreshTime }}</el-tag>
          </span>
        </el-form-item>
        <el-form-item :label="this.$t('common.auto')" style="float: right;">
          <el-switch v-model="form.auto" :disabled="disabled" @change="handlerAuto"></el-switch>
        </el-form-item>
        <el-form-item :label="this.$t('common.threshold')" style="float: right;">
          <el-input-number v-model="form.threshold" :disabled="disabled" :min="1" :max="10"></el-input-number>
        </el-form-item>
      </el-form>
    </el-row>
    <highcharts :options="chartOptions"></highcharts>
    <el-table v-if="getLengthGtZore(tableDatas)" :data="tableDatas" style="width: 100%">
      <el-table-column prop="time" :label="this.$t('common.time')"></el-table-column>
      <el-table-column prop="rows" :label="this.$t('common.rows')"></el-table-column>
      <el-table-column prop="elapsed" :label="this.$t('common.elapsed')"></el-table-column>
      <el-table-column prop="bytes" :label="this.$t('common.bytes')"></el-table-column>
      <el-table-column prop="memoryUsage" :label="this.$t('common.memoryUsage')"></el-table-column>
      <el-table-column prop="bytesRead" :label="this.$t('common.bytesRead')"></el-table-column>
      <el-table-column prop="bytesWritten" :label="this.$t('common.bytesWritten')"></el-table-column>
      <el-table-column prop="hash" :label="this.$t('common.hash')"></el-table-column>
      <el-table-column prop="host" :label="this.$t('common.host')"></el-table-column>
    </el-table>
  </div>
</template>

<script>
import DataSourceSelect from '@/views/components/data/datasource/DataSourceSelect'
import { getDataSources } from '@/services/DataSource'
import { getProcesses } from '@/services/Monitor'
import { buildArray } from '@/utils/ArrayUtils'

export default {
  components: {
    DataSourceSelect
  },
  data() {
    return {
      tableDatas: [],
      selectServerValue: null,
      timer: 0,
      refreshTime: 0,
      form: {
        threshold: 2,
        auto: false
      },
      disabled: true,
      dataCount: [],
      chartOptions: {
        title: {
          text: this.$t('common.processor') + this.$t('common.count')
        },
        credits: {
          enabled: false
        },
        series: [{
          name: this.$t('common.count'),
          data: []
        }]
      }
    }
  },
  created() {
    this._initialize()
  },
  methods: {
    _initialize() {
      this.selectServers = getDataSources(null).columns
    },
    async handlerServer(value) {
      this.selectServerValue = value
      const response = await getProcesses(this.selectServerValue)
      if (!response.status) {
        this.$notify.error({
          title: 'Error',
          message: response.message
        })
      } else {
        this.tableDatas = response.columns
      }
      this.disabled = false
      this.chartOptions.series[0].data = buildArray(this.dataCount, 20, true, this.tableDatas.length)
    },
    handlerAuto() {
      if (this.form.auto) {
        this.timer = setInterval(() => {
          this.handlerServer(this.selectServerValue)
          this.refreshTime = (new Date()).valueOf()
        }, this.form.threshold * 1000)
      } else {
        clearInterval(this.timer)
      }
    }
  },
  beforeDestroy() {
    if (this.timer) {
      clearInterval(this.timer)
    }
  },
  watch: {
    chartOptions: {}
  }
}
</script>
