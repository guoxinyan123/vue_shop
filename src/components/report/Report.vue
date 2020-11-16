<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>数据统计</el-breadcrumb-item>
        <el-breadcrumb-item>数据报表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <div id="main" style="width: 800px;height:400px;"></div>
      </el-card>
    </div>
</template>

<script>
import echarts from 'echarts'
import _ from 'lodash'
export default {
  data: function () {
    return {
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [{ boundaryGap: false }],
        yAxis: [{ type: 'value' }]
      }
    }
  },
  mounted: async function () {
    var myChart = echarts.init(document.getElementById('main'))
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) return this.$http.error('获取报表失败')
    const result = _.merge(res.data, this.options)
    myChart.setOption(result)
  }
}
</script>

<style lang="less" scoped>
</style>
