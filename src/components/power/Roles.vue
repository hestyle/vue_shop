<template>
  <div>
     <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', index1 === 0 ? 'bdtop': '', 'vcenter']" v-for="(item1, index1) in scope.row.children" :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag @close="deleteRightById(scope.row, item1.id)" closable>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级、三级权限 -->
              <el-col :span="19">
                <el-row :class="[index2 !== 0 ? 'bdtop': '', 'vcenter']" v-for="(item2, index2) in item1.children" :key="item2.id">
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" @close="deleteRightById(scope.row, item1.id)" closable>{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3, index3) in item2.children" :key="item3.id" @close="deleteRightById(scope.row, item2.id)" closable>{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="290">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="editRole(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="setRoleRights(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色 -->
    <el-dialog title="修改角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="70px">
        <!-- 角色名 -->
        <el-form-item prop="roleName" label="角色名">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <!-- 描述 -->
        <el-form-item prop="roleDesc" label="描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoleInfo">添 加</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClose">
      <el-form ref="editFormRef" :rules="editFormRules" :model="editForm" label-width="70px">
        <!-- 角色名 -->
        <el-form-item prop="roleName" label="角色名">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <!-- 描述 -->
        <el-form-item prop="roleDesc" label="描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">保 存</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色权限 -->
    <el-dialog title="分配权限" :visible.sync="setRightsDialogVisible" width="50%" @close="setRoleRightsDialogClose">
      <!-- 树形控件 -->
      <el-tree ref="rightsTreeRef" :data="rightsList" :props="rightsTreeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="rightsCheckedKeys"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">保 存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      // 控制添加role对话框是否显示
      addDialogVisible: false,
      addForm: {
        roleName: '',
        roleDesc: '',
      },
      // 表单验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '请输入添加角色名', trigger: 'blur' },
          {
            min: 2,
            max: 15,
            message: '长度在 2 到 15 个字符',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: true, message: '请添加角色描述', trigger: 'blur' },
          {
            min: 2,
            max: 50,
            message: '长度在 2 到 50 个字符',
            trigger: 'blur',
          },
        ]
      },
      // role edit
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        roleName: [
          { required: true, message: '请输入修改后的角色名', trigger: 'blur' },
          {
            min: 2,
            max: 15,
            message: '长度在 2 到 15 个字符',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: true, message: '请输入修改后的角色描述', trigger: 'blur' },
          {
            min: 2,
            max: 50,
            message: '长度在 2 到 50 个字符',
            trigger: 'blur',
          },
        ],
      },
      // 设置权限
      // role edit
      setRightsDialogVisible: false,
      settingRightsRoleId: '',
      rightsList: [],
      rightsTreeProps: {
        label: 'authName',
        children: 'children'
      },
      rightsCheckedKeys: [104, 115]
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    // 获取roleList
    async getRoleList() {
      const {data: res} = await this.$http.get('roles')
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.roleList = res.data
    },
    // add role
    addDialogClose() {
      // 重置editForm
      this.$refs.addFormRef.resetFields()
    },
    addRoleInfo() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const {data: res} = await this.$http.post('roles', this.addForm)
        console.log(res);
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加成功')
        }
        // 关闭窗口
        this.addDialogVisible = false
        // 刷新表格
        this.getRoleList();
      })
    },
    // edit role
    editRole(id) {
      this.editDialogVisible = true
      this.getRoleById(id)
    },
    // 通过id获取role
    async getRoleById(id) {
      const {data: res} = await this.$http.get('roles/' + id)
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
    },
    // 关闭编辑role对话框回调
    editDialogClose() {
      // 重置editForm
      this.$refs.editFormRef.resetFields()
    },
    // 保存修改
    async saveRoleInfo() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('表单存在非法字段')
        }
        // 请求
        const {data: res} = await this.$http.put('roles/' + this.editForm.roleId, {
          "roleName": this.editForm.roleName,
          "roleDesc": this.editForm.roleDesc
        })
        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('保存成功')
        }
        // 关闭窗口
        this.editDialogVisible = false
        // 刷新表格
        this.getRoleList();
      })
    },
    // 通过id删除role
    removeRoleById(id) {
      this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('roles/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('删除用户成功')
        }
        console.log(res)
        // 刷新table
        this.getRoleList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    // 通过id删除right
    deleteRightById(role, rightId) {
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('删除权限成功')
        }
        console.log(res)
        // 刷新table
        // this.getRoleList()
        role.children = res.data
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    // 设置权限
    async setRoleRights(role) {
      const {data: res} = await this.$http.get('rights/tree')
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightsList = res.data
      // (递归)获取三级权限
      this.rightsCheckedKeys = []
      this.getLeafKeys(role, this.rightsCheckedKeys)
      // 保存正在设置权限的roleId
      this.settingRightsRoleId = role.id

      this.setRightsDialogVisible = true
    },
    // (递归)获取三级权限
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      // node.children.forEach(element => {
      //   this.getLeafKeys(element, arr)
      // });
      for (let i = 0; i < node.children.length; ++i) {
        this.getLeafKeys(node.children[i], arr)
      }
    },
    // 关闭权限对话框时清空
    setRoleRightsDialogClose() {
      this.settingRightsRoleId = ''
      this.rightsCheckedKeys = []
    },
    // 保存角色分配的权限
    async allotRights() {
      const keys = [
        ...this.$refs.rightsTreeRef.getCheckedKeys(),
        ...this.$refs.rightsTreeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      
      const {data: res} = await this.$http.post(`roles/${this.settingRightsRoleId}/rights`, {rids: idStr})
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('权限分配成功')

      this.setRightsDialogVisible = false
      this.getRoleList()
    }
  },
}
</script>

<style>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>