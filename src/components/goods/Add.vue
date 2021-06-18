<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 提示区域 -->
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false"></el-alert>
      <!-- 步骤进度条区域 -->
      <el-steps :space="200" :active="parseInt(activeIndex)" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm" :rules="addFormRules" 
        ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave" @tab-click="tabClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item prop="goods_name" label="商品名称">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item prop="goods_price" label="商品价格/元">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item prop="goods_weight" label="商品重量/kg">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item prop="goods_number" label="商品数量/件">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item prop="goods_cat" label="商品分类">
              <el-cascader
                :options="categoryList" 
                :props="cascaderProps" 
                v-model="addForm.goods_cat"
                clearable
                @change="selectedCategoryChange">
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableData" :label="item.attr_name" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox v-for="(value, index) in item.attr_vals" :key="index" :label="value" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyTableData" :label="item.attr_name" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :headers="uploadHeaderObj"
              :on-success="uploadSuccess"
              list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="addButton" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImag">
    </el-dialog>

  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur'}
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur'}
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur'}
        ],
        goods_number: [
          { required: true, message: '请输入商品名数量', trigger: 'blur'}
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur'}
        ]
      },
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
      // 商品动态参数列表
      manyTableData: [],
      // 商品静态属性
      onlyTableData: [],
      // 上传图片的url
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      uploadHeaderObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewVisible: false,
      previewPath: ''
    }
  },
  created() {
    this.getCategoryList()
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
    // 级联选中项变更回调
    selectedCategoryChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.$message.error('只允许选择第三级商品分类')
        this.addForm.goods_cat = []
        return
      }
      this.getManyTableData()
      this.getOnlyTableData()
    },
    // 标签切换钩子函数
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        // 从基本信息tab跳转出需要判断是否选择了商品分类
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClicked() {
      if (this.activeIndex === '1') {
        
      } else if (this.activeIndex === '2') {
        
      }
    },
    // 获取动态参数
    async getManyTableData() {
      const {data: res} = await this.$http.get(`categories/${this.selectedCategoryId}/attributes`, { params: { sel: 'many' } })
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
      }
      this.manyTableData = res.data
    },
    // 获取静态属性
    async getOnlyTableData() {
      const {data: res} = await this.$http.get(`categories/${this.selectedCategoryId}/attributes`, { params: { sel: 'only' } })
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.onlyTableData = res.data
    },
    // 图片预览
    handlePreview(file) {
      console.log(file);
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 移除图片
    handleRemove(file) {
      console.log(file);
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(item => item.pic === filePath)
      this.addForm.pics.splice(index, 1)
      console.log(this.addForm)
    },
    // 图片上传成功
    uploadSuccess(response) {
      console.log(response);
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    addGoods() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('请填写完整的表单信息')
        }
        // 深拷贝
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          form.attrs.push(newInfo)
        });
        // 静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          form.attrs.push(newInfo)
        });
        console.log(form)
        // 请求
        const { data: res } = await this.$http.post(`goods`, form)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加成功')
        }
        // 关闭窗口
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    selectedCategoryId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      } else {
        return null
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 5px 0 0 !important;
}

.previewImag {
  width: 100%;
}

.addButton {
  margin: 15px 0px;
}
</style>