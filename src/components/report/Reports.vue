<template>
  <div>
      <!--面包屑导航-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图-->
      <el-card>
        <!-- 2为 ECharts 准备一个具备大小（宽高）的 DOM -->
        <div id="main" style="width: 750px;height:400px;"></div>
      </el-card>
  </div>
</template>

<script>
// 1导入echarts
import echarts from 'echarts'
// 7导入lodash用来做对象的合并
import _ from 'lodash'
export default {
  data () {
    return {
      // 6需要合并的数据
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
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  async mounted () {
    // 3基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'))
    // 4获取数据
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) return this.$message.error('数据获取失败')
    const result = _.merge(res.data, this.options)
    // 5使用刚指定的配置项和数据显示图表。
    myChart.setOption(result)
  },
  created () {},
  methods: {}
}
</script>

<style lang="less" scoped>
</style>
