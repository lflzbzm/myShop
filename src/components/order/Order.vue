<template>
  <div>
      <!--面包屑导航-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>订单管理</el-breadcrumb-item>
        <el-breadcrumb-item>订单列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图-->
      <el-card>
          <!--搜索框-->
          <el-row>
              <el-col :span="8">
                  <el-input placeholder="请输入内容">
                    <el-button slot="append" icon="el-icon-search"></el-button>
                  </el-input>
              </el-col>
          </el-row>
          <!--表格-->
          <el-table :data="orderList" border stripe>
            <el-table-column prop="order_number" label="订单编号" width="200px">
            </el-table-column>
            <el-table-column prop="order_price" label="订单价格">
            </el-table-column>
            <el-table-column prop="pay_status" label="是否付款">
                <template slot-scope="scope">
                    <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
                    <el-tag type="danger" v-else>未付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="is_send" label="是否发货">
            </el-table-column>
            <el-table-column prop="create_time" label="下单时间">
            </el-table-column>
            <el-table-column label="操作">
                <template>
                    <el-button @click="showEditDialog" size="mini" type="primary" icon="el-icon-edit"></el-button>
                    <el-button @click="showWuliuDialog" size="mini" type="success" icon="el-icon-location"></el-button>
                </template>
            </el-table-column>
          </el-table>
          <!--分页-->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[3, 5, 10, 20]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
      </el-card>
      <!--修改地址对话框-->
      <el-dialog
        title="修改地址"
        :visible.sync="editDialog"
        width="50%"
        @close="editDialogClosed">
        <!--表单-->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
            <el-form-item label="省市区/县" prop="address1">
                  <el-cascader
                    v-model="editForm.address1"
                    :options="cityData">
                  </el-cascader>
            </el-form-item>
            <el-form-item label="详细地址" prop="address2">
                <el-input></el-input>
            </el-form-item>
        </el-form>
        <!--底部-->
        <span slot="footer">
            <el-button @click="editDialog = false">取 消</el-button>
            <el-button type="primary" @click="editDialog = false">确 定</el-button>
        </span>
      </el-dialog>
      <!--物流信息对话框-->
      <el-dialog
        title="物流信息"
        :visible.sync="wuliuDialog"
        width="30%">
        <!--时间线-->
        <el-timeline>
            <el-timeline-item
            v-for="(activity, index) in wuliuInfo"
            :key="index"
            :timestamp="activity.timestamp">
            {{activity.context}}
            </el-timeline-item>
        </el-timeline>
        <span slot="footer">
            <el-button @click="wuliuDialog = false">取 消</el-button>
            <el-button type="primary" @click="wuliuDialog = false">确 定</el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      cityData,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      total: 0,
      orderList: [],
      editDialog: false,
      editForm: {
        address1: [],
        address2: []
      },
      editFormRules: {
        address1: [
          { required: true, message: '请选择收货地址', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      wuliuDialog: false,
      wuliuInfo: []
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取订单数据失败')
      console.log(res.data)
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getOrderList()
    },
    handleCurrentChange (newpagenum) {
      this.queryInfo.pagenum = newpagenum
      this.getOrderList()
    },
    showEditDialog () {
      this.editDialog = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async showWuliuDialog () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) return this.$message.error('获取物流信息失败')
      console.log(res.data)
      this.wuliuInfo = res.data
      this.wuliuDialog = true
    }
  }
}
</script>

<style lang="less" scoped>
</style>
