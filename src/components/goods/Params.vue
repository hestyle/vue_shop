<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告提示 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择器 -->
          <el-cascader
            :options="categoryList" 
            :props="cascaderProps" 
            v-model="selectedKeys"
            clearable
            @change="selectedCategoryChange">
          </el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
          <el-table :data="manyTabData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 作用域插槽 -->
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- 作用域插槽 -->
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
          <el-table :data="onlyTabData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 作用域插槽 -->
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <!-- 作用域插槽 -->
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加动态参数、静态属性 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="100px">
        <!-- 动态参数、静态属性 -->
        <el-form-item prop="attr_name" :label="titleText">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改动态参数、静态属性 -->
    <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClose">
      <el-form ref="editFormRef" :rules="editFormRules" :model="editForm" label-width="100px">
        <!-- 动态参数、静态属性 -->
        <el-form-item prop="attr_name" :label="titleText">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import itemVue from '../../../../17-21 Vue.js项目实战开发/20-21vue电商/8.vue-项目实战day6/code/vue_shop/src/plugins/timeline/src/item.vue'
export default {
  data() {
    return {
      categoryList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: false
      },
      // 选中的父级分类
      selectedKeys: [],
      // 选中的tab
      activeName: 'many',
      // 动态参数
      manyTabData: [],
      // 静态属性
      onlyTabData: [],
      // 添加动态参数、静态属性
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: `请输入添加的名称`, trigger: 'blur' }
        ]
      },
      // 编辑动态参数、静态属性
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: `请输入修改后的名称`, trigger: 'blur' }
        ]
      },
      

    }
  },
  created() {
    // 获取所有商品分类
    this.getCategoryList()
  },
  computed: {
    isBtnDisabled() {
      // 只有当选中了三级分类，添加参数、属性按钮才可点击
      return this.selectedKeys.length !== 3
    },
    selectedCategoryId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      } else {
        return null
      }
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },
  methods: {
    async getCategoryList() {
      const {data: res} = await this.$http.get('categories')
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.categoryList = res.data
    },
    // 级联选择器变化回调函数
    selectedCategoryChange() {
      // 如果选中的不是三级分类，清空
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTabData = []
        this.onlyTabData = []
        return
      }
      // 获取tab所处面板的参数
      this.getParamsData()
    },
    // tab切换回调
    handleTabClick() {
      // 获取tab所处面板的参数
      this.getParamsData()
    },
    // 获取tab所处面板的参数
    async getParamsData() {
      if (this.selectedKeys.length !== 3) {
        return
      }
      const {data: res} = await this.$http.get(`categories/${this.selectedCategoryId}/attributes`, { params: { sel: this.activeName } })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      for (let i = 0; i < res.data.length; ++i) {
        if (res.data[i].attr_vals.length !== 0) {
          res.data[i].attr_vals = res.data[i].attr_vals.split(' ')
        } else {
          res.data[i].attr_vals = []
        }
        // 每一个都设置inputVisible、inputValue
        // 控制按钮与文本框的显示
        res.data[i].inputVisible = false
        res.data[i].inputValue = ''
      }
      // 保存获取的结果
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    addParams() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const { data: res } = await this.$http.post(`categories/${this.selectedCategoryId}/attributes`, {
          attr_name: this.addForm.attr_name, 
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加成功')
        }
        console.log(res)
        // 关闭窗口
        this.addDialogVisible = false
        // 刷新table
        this.getParamsData()
      })
    },
    async showEditDialog(attr_id) {
      const {data: res} = await this.$http.get(`categories/${this.selectedCategoryId}/attributes/${attr_id}`, { params: { sel: this.activeName } })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 保存获取的结果
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editParams() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const { data: res } = await this.$http.put(`categories/${this.selectedCategoryId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name, 
          attr_sel: this.activeName
        })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('编辑成功')
        }
        // 关闭窗口
        this.editDialogVisible = false
        // 刷新table
        this.getParamsData()
      })
    },
    removeParam(attr_id) {
      this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`categories/${this.selectedCategoryId}/attributes/${attr_id}`)
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('删除参数成功')
        }
        // 刷新table
        this.getParamsData()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    showInput(row) {
      row.inputVisible = true
      // （nextTick的作用等待页面元素被重新渲染之后）让文本框自动获取焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    async handleInputConfirm(row) {
      // 切换回tag
      let realValue = row.inputValue.trim()
      row.inputVisible = false
      row.inputValue = ''
      if (realValue.length === 0) {
        return
      }
      // 保存tag
      row.attr_vals.push(realValue)
      this.updateParams(row)
    },
    handleClose(index, row) {
      row.attr_vals.splice(index, 1)
      this.updateParams(row)
    },
    async updateParams(row) {
      const { data: res } = await this.$http.put(`categories/${this.selectedCategoryId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.$message.success('参数保存成功')
      }
    }
  },
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0px;
}

.input-new-tag {
  width: 120px;
}
</style>