<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCategoryDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table class="treeTable" :data="categoryList" 
        :columns="columns" 
        :selection-type="false" 
        :expand-type="false" 
        show-index index-text="#" 
        border :show-row-hover="false">
        <!-- 是否有效template -->
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!-- 排序template -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作template -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCategoryDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCategoryById(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类 -->
    <el-dialog title="添加分类" :visible.sync="addCategoryDialogVisible" width="50%" @close="addCategoryDialogClose">
      <el-form ref="addCategoryFormRef" :rules="addCategoryFormRules" :model="addCategoryForm" label-width="100px">
        <!-- 分类名称 -->
        <el-form-item prop="cat_name" label="分类名称：">
          <el-input v-model="addCategoryForm.cat_name"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item label="父级分类：">
          <el-cascader
            :options="parentCategoryList" 
            :props="cascaderProps" 
            v-model="selectedKeys"
            clearable
            @change="parentCategoryChange">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCategoryDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">保 存</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类 -->
    <el-dialog title="编辑分类" :visible.sync="editCategoryDialogVisible" width="50%" @close="editCategoryDialogClose">
      <el-form ref="editCategoryFormRef" :rules="editCategoryFormRules" :model="editCategoryForm" label-width="100px">
        <!-- 分类名称 -->
        <el-form-item prop="cat_name" label="分类名称：">
          <el-input v-model="editCategoryForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCategoryDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCategory">保 存</el-button>
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
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // category list
      categoryList: [],
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isOk'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      addCategoryDialogVisible: false,
      addCategoryForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCategoryFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur'}
        ]
      },
      // 父级分类（1,2级）
      parentCategoryList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选中的父级分类
      selectedKeys: [],
      // 编辑分类
      editCategoryDialogVisible: false,
      editCategoryForm: {
        cid: 0,
        cat_name: ''
      },
      editCategoryFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur'}
        ]
      },
    }
  },
  created() {
    this.getCategoryList()
  },
  methods: {
    async getCategoryList() {
      const {data: res} = await this.$http.get('categories', { params: this.queryInfo })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.categoryList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCategoryList()
    },
    handleCurrentChange(currentPage) {
      this.queryInfo.pagenum = currentPage
      this.getCategoryList()
    },
    showAddCategoryDialog() {
      // 获取parent category
      this.getParentCategoryList()
      this.addCategoryDialogVisible = true
    },
    async getParentCategoryList() {
      const {data: res} = await this.$http.get('categories', { params: { type: 2 } })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.parentCategoryList = res.data
    },
    parentCategoryChange() {
      if (this.selectedKeys.length > 0) {
        // 更新父级id、分类级别
        this.addCategoryForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCategoryForm.cat_level = this.selectedKeys.length
      } else {
        this.addCategoryForm.cat_pid = 0
        this.addCategoryForm.cat_level = 0
      }
      console.log(this.addCategoryForm);
    },
    // 监听对话框关闭事件
    addCategoryDialogClose() {
      // 清空数据
      this.$refs.addCategoryFormRef.resetFields()
      this.selectedKeys = []
      this.addCategoryForm.cat_level = 0
      this.addCategoryForm.cat_pid = 0
    },
    addCategory() {
      // 表单预验证
      this.$refs.addCategoryFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const { data: res } = await this.$http.post('categories', this.addCategoryForm)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加分类成功')
        }
        console.log(res)
        // 关闭窗口
        this.addCategoryDialogVisible = false
        // 刷新table
        this.getCategoryList()
      })
    },
    // 显示编辑对话框
    async showEditCategoryDialog(id) {
      // 通过id请求category数据
      const { data: res } = await this.$http.get('categories/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 暂存信息
      this.editCategoryForm.cid = id
      this.editCategoryForm.cat_name = res.data.cat_name

      this.editCategoryDialogVisible = true
    },
    // 监听对话框关闭事件
    editCategoryDialogClose() {
      // 清空数据
      this.$refs.editCategoryFormRef.resetFields()
    },
    editCategory() {
      // 表单预验证
      this.$refs.editCategoryFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const { data: res } = await this.$http.put('categories/' + this.editCategoryForm.cid, { cat_name: this.editCategoryForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('分类修改保存成功')
        }
        console.log(res)
        // 关闭窗口
        this.editCategoryDialogVisible = false
        // 刷新table
        this.getCategoryList()
      })
    },
    removeCategoryById(id) {
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('categories/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('删除分类成功')
        }
        console.log(res)
        // 刷新table
        this.getCategoryList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
</style>