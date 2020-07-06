<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
            <!-- 输入框 -->
            <el-row :gutter="20">
                <!-- 搜索框 -->
                <el-col :span="9">
                    <el-input placeholder="请输入内容" v-model="ordersKeyword" clearable @clear='getOrdersList'>
                        <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
                    </el-input>
                </el-col>
            </el-row>
            <!-- 商品列表表格 -->
            <el-table :data="ordersData" border stripe size="small">
                <el-table-column type="index" align="center"></el-table-column>
                <el-table-column prop="order_number" label="订单编号" ></el-table-column>
                <el-table-column prop="order_price" label="订单价格"  align="center"></el-table-column>
                <el-table-column  label="是否付款" align="center">
                    <template slot-scope="scope">
                        <el-tag type="danger" v-if="scope.row.pay_status == 0 ? true : false">未付款</el-tag>
                        <el-tag type="success" v-else>已付款</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="is_send" label="是否发货" align="center"></el-table-column>
                <el-table-column prop="create_time" label="下单时间" align="center"></el-table-column>
                <el-table-column label="操作" align="center">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="editAddress"></el-button>
                        <el-button type="success" icon="el-icon-location" size="mini" @click="showExpress"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="10"
                layout="total, sizes, prev, pager, next, jumper"
                :total="ordersTotal">
            </el-pagination>
        </el-card>
        <!-- 修改地址对话框 -->
        <el-dialog
            title="修改地址"
            :visible.sync="editDialogVisible"
            width="50%">
            <el-form :model="ordersRuleForm" :rules="ordersRules" ref="ordersRuleForm" label-width="100px">
                <el-form-item label="省市区/县" prop="city">
                    <el-cascader
                        expand-trigger='hover'
                        :options="city_data"
                        :value="ordersRuleForm.city"
                        @change="changeProvince"
                        change-on-select

                    >
                    </el-cascader>
                </el-form-item>
                <el-form-item label="详细地址" prop="street">
                    <el-input v-model="ordersRuleForm.street"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editDialogVisible = false">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 物流信息对话框 -->
        <el-dialog
            title="物流信息"
            :visible.sync="expressDialogVisible"
            width="50%">
            <el-timeline :reverse="true">
                <el-timeline-item
                v-for="(express, index) in expressInfo"
                :key="index"
                :timestamp="express.ftime">
                {{express.context}}
                </el-timeline-item>
            </el-timeline>
        </el-dialog>
    </div>
</template>
<script>
import city_data from './city_data.js'
export default {
  created () {
    this.getOrdersList()
  },
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      ordersTotal: '',
      ordersData: [],
      ordersKeyword: '',
      ordersRuleForm: {
        city: [],
        street: ''
      },
      editDialogVisible: false,
      city_data,
      ordersRules: {
        city: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        street: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      expressDialogVisible: false,
      expressInfo: []
    }
  },
  methods: {
    async getOrdersList () {
      console.log('ok')
      this.queryInfo.query = this.ordersKeyword
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) this.$message('获取订单信息失败')
      res.data.goods.forEach((item, index) => {
        item.create_time = new Date(item.create_time).toLocaleString()
        item.update_time = new Date(item.update_time).toLocaleString()
      })
      this.ordersTotal = res.data.total
      this.ordersData = res.data.goods
    },
    handleSizeChange (val) {
      this.queryInfo.pagesize = val
      this.getOrdersList()
    },
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getOrdersList()
    },
    editAddress () {
      this.editDialogVisible = true
    },
    async showExpress () {
      this.expressDialogVisible = true
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) this.$message('获取物流信息失败！')
      this.expressInfo = res.data
    }

  }
}
</script>
<style lang="less" scoped>
    .el-cascader {
        width: 100%;
    }
</style>
