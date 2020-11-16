<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>订单管理</el-breadcrumb-item>
        <el-breadcrumb-item>订单列表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-row>
          <el-col :span="8">
            <el-input placeholder="请输入内容">
              <el-button slot="append" icon="el-icon-search"></el-button>
            </el-input>
          </el-col>
        </el-row>
        <el-table :data="orderList" style="width: 100%" border stripe>
          <el-table-column type="index" label="#" width="120"></el-table-column>
          <el-table-column prop="order_number" label="订单编号"></el-table-column>
          <el-table-column prop="order_price" label="订单价格"></el-table-column>
          <el-table-column prop="pay_status" label="是否付款">
            <template slot-scope="scope">
              <el-tag size="mini" type="danger" v-if="scope.row.pay_status === '0'">未付款</el-tag>
              <el-tag size="mini" type="success" v-else>已付款</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="is_send" label="是否发货"></el-table-column>
          <el-table-column prop="create_time" label="下单时间">
            <template slot-scope="scope">{{scope.row.create_time | dateFormat('yyyy-MM-dd hh:mm:ss')}}</template>
          </el-table-column>
          <el-table-column label="操作">
            <template>
              <el-button size="mini" type="primary" icon="el-icon-edit" @click="showAddressBox"></el-button>
              <el-button size="mini" type="success" icon="el-icon-location" @click="showProgressBox"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[5, 10, 15, 20]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </el-card>
      <el-dialog @close="closeAddressDialog" title="修改地址" :visible.sync="addressDialog" width="50%">
        <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              :props="{ expandTrigger: 'hover', label: 'label', value: 'value', children: 'children' }"
              v-model="addressForm.address1" :options="citydata">
            </el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addressDialog = false">取 消</el-button>
          <el-button type="primary" @click="addressDialog = false">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog title="物流进度" :visible.sync="progressDialog" width="50%">
        <el-timeline>
          <el-timeline-item
            v-for="(activity, index) in progressInfo"
            :key="index"
            :timestamp="activity.time">
            {{activity.context}}
          </el-timeline-item>
        </el-timeline>
      </el-dialog>
    </div>
</template>

<script>
import citydata from './citydata'
export default {
  data: function () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      orderList: [],
      total: 0,
      addressDialog: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      citydata,
      progressDialog: false,
      progressInfo: []
    }
  },
  created: function () {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取订单列表失败')
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getOrderList()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getOrderList()
    },
    showAddressBox() {
      this.addressDialog = true
    },
    closeAddressDialog() {
      console.log(this)
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox() {
      this.progressDialog = true
      const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取物流信息失败')
      this.progressInfo = res.data
    }
  }
}
</script>

<style lang="less" scoped>
  .el-cascader {
    width: 100%;
  }
</style>
