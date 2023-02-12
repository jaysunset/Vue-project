<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 图像区域 -->
      <div class="avatar_box">
        <img src=".././assets/images/maer.jpg" alt="" />
      </div>
      <!-- 登入表单区域  -->
      <el-form
        ref="loginFormRef"
        label-width="0px"
        class="login_form"
        :model="loginFrom"
        :rules="rules"
      >
        <el-form-item prop="username">
          <el-input
            placeholder="请输入账号 "
            class="login_input"
            prefix-icon="el-icon-user"
            v-model="loginFrom.username"
          ></el-input>

          <!-- <el-lable class="login_lable">111111111</el-lable> -->
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            placeholder="请输入密码"
            prefix-icon="el-icon-lock"
            v-model="loginFrom.password"
            type="password"
          ></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登入</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 这是登入表单的数据绑定对象
      loginFrom: {
        username: "",
        password: ""
      },
      // 这是表单验证规则对象 
      rules: {
        // 验证用户名是否合法 (与上面绑定的数据名称要一致)
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 2, max: 5, message: '长度在 2 到 5 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 10, message: '长度在 5 到 10 个字符', trigger: 'blur' }

        ]
      }
    }
  },
  methods: {
    //点击重置按钮，重置登录表单
    resetLoginForm() {
      this.$refs.loginFormRef.resetFields();
    },

    login() {
      this.$refs.loginFormRef.validate(async valid => {//进行async await 异步处理
        if (!valid) return;  //对整个表单进行校验 成功则发送请求
        const { data: res } = await this.$http.post('login', this.loginFrom)
        if (res.meta.status !== 200) return this.$message.error("登录失败");
        this.$message.success("登录成功")
        // 1.将登录成功之后的 token 保存到客户端 的 sessionstorage中 
        //1.1 项目中除了登录页面外 其他页面Api接口都需要登录之后才能访问
        //1.2 token只应在当前网站打开期间生效 所以应该保存在 sessionstora中
        window.sessionStorage.setItem("token", res.data.token)
        //2. 通过 编程式导航 跳转到后台主页 路由地址为 /home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style  scoped>
.login_container {
  height: 100%;
  background-color: skyblue;
  /* display: flex;
  justify-content: center;
  align-items: center; */
}
.login_box {
  width: 30%;
  height: 40%;
  background-color: #fff;
  border-radius: 5px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  /* animation: borderAnimation 3s ease-in; */
}

.avatar_box img {
  height: 80px;
  width: 80px;
  border-radius: 50%;
  border: 10px solid skyblue;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: all 1s ease-in-out;
}

.avatar_box img:hover {
  height: 100px;
  width: 100px;
  border-radius: 50%;
  border: 0px solid skyblue;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: all 1s ease-in-out;
}
.login_form {
  width: 90%;
  position: absolute;
  bottom: 0px;
  margin: 0 5%;
}
.btns {
  display: flex;
  justify-content: center;
  align-items: center;
}
/* .login_lable {
  position: absolute;
  margin-top: 0px;
  margin-left: -95%;
}
.login_input:hover ~ .login_lable {
  margin-top: -20px;
} */
</style>