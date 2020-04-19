<template>
  <div class="login_container">
    <div class="login_box">
      <!--头像区域-->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!--表单区域-->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRoles"
               label-width="0px" class="login_form">
        <!--账号-->
        <el-form-item prop="phone">
          <el-input v-model="loginForm.phone" placeholder="请输入注册手机号"
                    prefix-icon="el-icon-user"></el-input>
        </el-form-item>
        <!--密码-->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" placeholder="请输入密码"
                    prefix-icon="el-icon-lock" type="password"></el-input>
        </el-form-item>
        <!--按钮区域-->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>

    </div>
  </div>
</template>

<script>

  export default {
    data () {
      return {
        //登录表单数据绑定对象
        loginForm: {
          phone: '',
          password: ''
        },
        //表单验证规则
        loginFormRoles: {
          //验证用户名是否符合规则
          phone: [
            {
              required: true,
              message: '请输入注册手机号',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 6,
              message: '长度在 3 到 6 个字符',
              trigger: 'blur'
            }
          ],
          //验证密码是否符合规则
          password: [
            {
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 16,
              message: '长度在 3 到 16 个字符',
              trigger: 'blur'
            }
          ]
        }
      }
    },
    methods: {
      //重置事件
      resetLoginForm () {
        // console.log(this)
        this.$refs.loginFormRef.resetFields()
      },

      login () {
        //表单预验证
        this.$refs.loginFormRef.validate(async valid => {
          console.log(valid)
          if (!valid) return
          const { data: res} = await this.$http.post("/login/login", this.loginForm)

          if(res.code != 200) return this.$message.error('登陆失败！')
          this.$message.success('登陆成功！')
          //登陆成功之后的token保存到客户端的sessionStorage中
          // window.sessionStorage.setItem('token',res.data.token)

          this.$router.push('/home')
        })
      }
    }
  }
</script>

<style lang="less" scoped>
  .login_container {
    background-color: #2b4b6b;
    height: 100%;
  }

  .login_box {
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    .avatar_box {
      height: 130px;
      width: 130px;
      border: 1px solid #eee;
      border-radius: 50%; /*圆角*/
      padding: 10px; /*内边距*/
      box-shadow: 0 0 10px #ddd; /*阴影*/
      position: absolute;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: #fff;
      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #eee;
      }
    }
  }

  .login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
  }

  .btns {
    display: flex;
    justify-content: flex-end;
  }

</style>
