<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 登入图标 -->
      <div class="avatar_box">
        <img src=".././assets/logo.png" alt="" />
      </div>
      <!-- 表单 -->
      <el-form
        :model="ruleForm"
        :rules="rules"
        ref="ruleForm"
        class="login-ruleForm"
      >
        <!-- 有户名输入框 -->
        <el-form-item prop="username">
          <el-input v-model="ruleForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码输入框 -->
        <el-form-item prop="password">
          <el-input v-model="ruleForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item class="btn">
          <el-button type="primary" @click="login">登入</el-button>
          <el-button type="info" @click="resetFields">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      //表单数据
      ruleForm: {
        username: 'admin',
        password: '123456',
      },
      //表单验证规则
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 6, message: '长度在 3 到 6 个字符', trigger: 'blur' },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  mounted() {
    //监听回车登入
    window.addEventListener('keydown', this.keyDown)
  },
  methods: {
    //重置表单
    resetFields() {
      this.$refs.ruleForm.resetFields()
    },
    //点击按钮登入功能
    login() {
      this.$refs.ruleForm.validate(async (vaild) => {
        //未通过验证
        if (!vaild) return
        //通过验证，发送登入请求,获取返回信息
        var { data: res } = await this.$http.post('login', this.ruleForm)
        //用户名或者密码错误时
        if (res.meta.status !== 200)
          return this.$message.error('密码或用户名错误')
        //登入成功，保存token，并跳转至Home
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    },
    //回车登入功能
    keyDown(e) {
      //如果是回车则执行登录方法
      if (e.keyCode == 13) {
        this.login()
      }
    }
  },
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  height: 300px;
  width: 450px;
  background-color: #fff;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.avatar_box {
  width: 130px;
  height: 130px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 50%;
  background-color: #fff;
  box-shadow: 0 0 10px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
.login-ruleForm {
  padding: 0 22px;
  margin: 114px 0 0 0;
  .btn {
    display: flex;
    justify-content: flex-end;
  }
}
</style>