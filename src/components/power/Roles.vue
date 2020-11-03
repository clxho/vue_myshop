<template>
  <div>
    <!-- 面包屑导航区域 start-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 面包屑导航区域 end-->
    <!-- 卡片区 start -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-button type="primary" @click="showAddRoleDialog">添加角色</el-button>
      <!-- 角色列表 strat -->
      <el-table :data="rolesList" border style="width: 100%">
        <!-- 展开部分 start -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item, index) in scope.row.children"
              :key="index"
              :class="['bdbottom', index === 0 ? 'bdtop' : '']"
              class="vxenter"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeTagById(scope.row, item.id)">
                  {{ item.authName }}
                </el-tag>
                <i class="el-icon-arrow-right"></i>
              </el-col>
              <!-- 二级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(subitem, index) in item.children"
                  :key="index"
                  :class="[index === 0 ? '' : 'bdtop']"
                  class="vxenter"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeTagById(scope.row, subitem.id)"
                    >
                      {{ subitem.authName }}
                    </el-tag>
                    <i class="el-icon-arrow-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      v-for="(ssubitem, index) in subitem.children"
                      :key="index"
                      type="warning"
                      closable
                      @close="removeTagById(scope.row, ssubitem.id)"
                    >
                      {{ ssubitem.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 展开部分 end -->
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini"
              >编辑</el-button
            >
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteRole(scope.row.id)"
              >删除</el-button
            >
            <!-- 分配权限按钮 -->
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialong(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
        <!-- 角色列表 end -->
      </el-table>
      <!-- 添加角色对话框 start -->
      <el-dialog
        title="提示"
        :visible.sync="addRoleDialogVisible"
        width="50%"
        @close="addRoleDialogVisibleClosed"
      >
        <!-- 角色对话框主体区 -->
        <el-form
          ref="addRoleFormRef"
          :model="addRoleForm"
          :rules="addRoleFormRules"
          label-width="70px"
        >
          <el-form-item label="名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="描述" prop="roleDesc">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 角色对话框底部 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加角色对话框 end -->
      <!-- 分配权限对话框 start -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
        @close="setRightDialogClosed"
      >
        <el-tree
          :data="rightList"
          :props="treeProps"
          show-checkbox
          default-expand-all
          node-key="id"
          :default-checked-keys="defKeys"
          ref="treeRef"
        ></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配权限对话框 end -->
    </el-card>
    <!-- 卡片区 end -->
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      //角色列表
      rolesList: [],
      addRoleDialogVisible: false,
      //添加角色数据
      addRoleForm: {
        roleName: '',
        roleDesc: '',
      },
      //添加角色验证规则
      addRoleFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur',
          },
        ],
      },
      setRightDialogVisible: false,
      //将要分配权限的id
      roleId: '',
      //权限列表
      rightList: [],
      treeProps: {
        children: 'children',
        label: 'authName',
      },
      defKeys: [],
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    //后去角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      this.rolesList = res.data
    },
    //显示添加角色对话框
    showAddRoleDialog() {
      this.addRoleDialogVisible = true
    },
    //添加角色功能
    async addRole() {
      const { data: res } = await this.$http.post('roles', {
        roleName: this.addRoleForm.roleName,
        roleDesc: this.addRoleForm.roleDesc,
      })
      if (res.meta.status !== 201) {
        return this.$message.error('添加角色失败')
      }
      this.addRoleForm = res.data
      this.addRoleDialogVisible = false
      this.$message.success('添加角色成功')
      this.getRolesList()
    },
    //添加角色对话框关闭时 重置表单
    addRoleDialogVisibleClosed() {
      this.$refs.addRoleFormRef.resetFields()
    },
    //删除角色
    async deleteRole(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      //如果用户点击取消,则返回值为cancel
      //如果用户点击确定，则返回字符串
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除')
      }

      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status != 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getRolesList()
    },
    //删除权限标签
    async removeTagById(role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((error) => error)
      if (confirmResult != 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status != 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    //显示分配权限对话框 并展示数据
    async showSetRightDialong(role) {
      //存储需要分配权限的id
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status != 200) {
        return this.$message.error('获取权限数据失败')
      }
      this.rightList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    //获取角色所有三级权限的id
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach((item) => {
        this.getLeafKeys(item, arr)
      })
    },
    //分配用户对话框关闭清空 defKeys数组
    setRightDialogClosed() {
      this.defKeys = []
    },
    //分配权限提交
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status != 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
  },
}
</script>

<style lang="less" scoped>
.el-form-item__label {
  padding: 0 !important;
}

.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vxenter {
  display: flex;
  align-items: center;
}
</style>