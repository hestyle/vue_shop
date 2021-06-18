<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getOrderList">
            <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格/元" prop="order_price" width="130px"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="130px">
          <template slot-scope="scope">
            <el-tag type="danger" size="small" v-if="scope.row.pay_status === '0'">未付款</el-tag>
            <el-tag type="success" size="small" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" width="130px"></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="160px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="addressDialogVisible = true"></el-button>
            <el-button type="warning" icon="el-icon-location-outline" size="mini" @click="showExpressDialog"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total" background>
      </el-pagination>
    </el-card>

    <!-- 修改地址 -->
    <el-dialog title="修改" :visible.sync="addressDialogVisible" width="50%">
      <el-form ref="addressFormRef" :rules="addressFormRules" :model="addressForm" label-width="100px">
        <!-- 商品名称 -->
        <el-form-item prop="address1" label="省市区县">
          <el-cascader
            :options="cityData" 
            :props="cascaderProps" 
            v-model="addressForm.address1"
            clearable
            @change="selectedCategoryChange">
          </el-cascader>
        </el-form-item>
        <!-- 商品价格 -->
        <el-form-item prop="address2" label="详细地址">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressDialogVisible = false">保 存</el-button>
      </span>
    </el-dialog>

    <!-- 查看物流信息 -->
    <el-dialog title="物流进度" :visible.sync="expressDialogVisible" width="50%">
      <el-timeline>
        <el-timeline-item
          v-for="(expressInfo, index) in expressInfos"
          :key="index"
          :timestamp="activity.ftime">
          {{expressInfo.content}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      // 分页查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      orderList: [],
      total: 0,
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: `请输入详细地址`, trigger: 'blur' }
        ],
        address2: [
          { required: true, message: `请输入详细地址`, trigger: 'blur' }
        ]
      },
      expressDialogVisible: false,
      expressInfos: [],
      cityData: cityData,
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'value',
        label: 'label',
        children: 'children',
        checkStrictly: false
      },
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const {data: res} = await this.$http.get('orders', { params: this.queryInfo })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPageNum) {
      this.queryInfo.pagenum = newPageNum
      this.getOrderList()
    },
    selectedCategoryChange() {
      console.log(this.addressForm);
    },
    async showExpressDialog() {
      const {data: res} = await this.$http.get('/kuaidi/1106975712662')
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.expressInfos = res.data
      this.expressDialogVisible = true
    }
  },
}
</script>

<style lang="less" scoped>

</style>