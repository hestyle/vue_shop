<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="gotoAddPage()">添加商品</el-button>
        </el-col>
      </el-row>
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格/元" prop="goods_price" width="95px"></el-table-column>
        <el-table-column label="商品重量/kg" prop="goods_weight" width="95px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showGoodsEditDialog(scope.row.goods_id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteByGoodsId(scope.row.goods_id)"></el-button>
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

    <!-- 编辑商品 -->
    <el-dialog title="修改商品信息" :visible.sync="editDialogVisible" width="50%" @close="editDialogClose">
      <el-form ref="editFormRef" :rules="editFormRules" :model="editForm" label-width="110px">
        <!-- 商品名称 -->
        <el-form-item prop="goods_name" label="商品名称">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <!-- 商品价格 -->
        <el-form-item prop="goods_price" label="商品价格/元">
          <el-input v-model="editForm.goods_price" type="number"></el-input>
        </el-form-item>
        <!-- 商品重量 -->
        <el-form-item prop="goods_weight" label="商品重量/kg">
          <el-input v-model="editForm.goods_weight" type="number"></el-input>
        </el-form-item>
        <!-- 库存 -->
        <el-form-item prop="goods_number" label="商品库存/件">
          <el-input v-model="editForm.goods_number" type="number"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGoodsInfo">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 分页查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0,
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品库存', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
      },
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const {data: res} = await this.$http.get('goods', { params: this.queryInfo })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPageNum) {
      this.queryInfo.pagenum = newPageNum
      this.getGoodsList()
    },
    deleteByGoodsId(goodsId) {
      this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('goods/' + goodsId)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('删除商品成功')
        }
        console.log(res)
        // 刷新table
        this.getGoodsList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    gotoAddPage() {
      this.$router.push('/goods/add')
    },
    async showGoodsEditDialog(goodsId) {
      const { data: res } = await this.$http.get('goods/' + goodsId)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.editForm = {}
    },
    editGoodsInfo() {
      // 表单预验证
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const { data: res } = await this.$http.put('goods/' + this.editForm.goods_id, this.editForm)
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('修改商品信息成功')
        }
        // 关闭窗口
        this.editDialogVisible = false
        // 刷新table
        this.getGoodsList()
      })
    }
  },
}
</script>

<style lang="less" scoped>

</style>