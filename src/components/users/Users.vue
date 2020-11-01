<template>
  <div>
    <!-- 面包屑 start -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 面包屑 end -->
    <!-- 卡片区 start -->
    <el-card>
      <!-- 搜索与添加区域 start -->
      <el-row :gutter="20">
        <!-- 搜索框 -->
        <el-col :span="10">
          <el-input
            placeholder="请输入内容"
            class="input-with-select"
            v-model="queryInfo.query"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList()"
            ></el-button> </el-input
        ></el-col>
        <el-col :span="4">
          <el-button type="primary" @click="showAddDialog">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 搜索与添加区域 end -->
      <!-- 用户列表 start -->
      <el-table :data="userList" border>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名" width="180">
        </el-table-column>
        <el-table-column prop="email" label="邮箱" width="180">
        </el-table-column>
        <el-table-column prop="mobile" label="电话" width="180">
        </el-table-column>
        <el-table-column prop="role_name" label="角色" width="180">
        </el-table-column>
        <el-table-column label="状态" width="180">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="250">
          <template slot-scope="scope">
            <!-- 编辑用户按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除用户按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteUser(scope.row.id)"
            ></el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 用户列表 end -->
      <!-- 添加用户对话框 start -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!--表单区  -->
        <el-form
          ref="addFormRef"
          :model="addForm"
          label-width="80px"
          :rules="addFormRules"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加用户对话框 end -->
      <!-- 修改用户对话框 start -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <!--表单区  -->
        <el-form
          ref="editFormRef"
          :model="editForm"
          label-width="80px"
          :rules="editFormRules"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户对话框 end -->
      <!-- 分页 start -->
      <el-pagination
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 分页 end -->
    </el-card>
    <!-- 卡片区 end -->
  </div>
</template>

<script>
export default {
  name: 'User',
  data() {
    //验证邮箱的规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      if (regEmail.test(value)) {
        //如果邮箱合法
        return callback()
      }
      callback(new Error('请输入合法邮箱'))
    }
    //验证手机号的规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[3|4|5|8｜9][0-9]\d{4,8}$/
      if (regMobile.test(value)) {
        //如果邮箱合法
        return callback()
      }
      callback(new Error('请输入合法手机号'))
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2,
      },
      total: 0,
      userList: [],
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: '',
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur',
          },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur',
          },
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            validator: checkEmail,
            trigger: 'blur',
          },
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            validator: checkMobile,
            trigger: 'blur',
          },
        ],
      },
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            validator: checkEmail,
            trigger: 'blur',
          },
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            validator: checkMobile,
            trigger: 'blur',
          },
        ],
      },
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    //获取用户列表数据函数
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo,
      })
      if (res.meta.status != 200) return this.$message.error('获取用户列表失败')
      this.userList = res.data.users
      this.total = res.data.total
    },
    //监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    //监听pagenum改变
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getUserList()
    },
    //监听用户状态改变
    async userStateChanged(userInfo) {
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      if (res.meta.status != 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    //显示添加用户对话框
    showAddDialog() {
      this.addDialogVisible = true
    },
    //重置添加用户对话框
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //添加用户功能
    addUser() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        //发起网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        if (res.meta.status != 201) return this.$message.error('添加用户失败')

        this.$message.success('添加用户成功')
        //隐藏添加用户对话框
        this.addDialogVisible = false
        //重新获取用户列表数据
        this.getUserList()
      })
    },
    //显示修改改用户对话框,并根据id将信息显示在修改用户对话框内
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      this.editForm = res.data
    },
    //修改用户信息功能
    editUser() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        //发起网络请求
        const { data: res } = await this.$http.put(
          'users/' + this.editForm.id,
          {
            email: this.editForm.email,
            mobile: this.editForm.mobile,
          }
        )
        if (res.meta.status != 200) return this.$message.error('修改用户失败')
        this.$message.success('修改用户成功')
        //隐藏修改用户对话框
        this.editDialogVisible = false
        //重新获取用户列表数据
        this.getUserList()
      })
    },
    //重置添加用户对话框
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //删除用户功能
    async deleteUser(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
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

      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status != 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getUserList()
    },
  },
}
</script>

<style lang="less" scoped>
</style>