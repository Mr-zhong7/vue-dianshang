<template>
  <div class="login_container">

    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/login.jpg" alt="">
      </div>
      <!-- 登录表单区域 -->
      <el-form :model="loginForm" ref="loginFormRef" :rules="loginFormRules" label-width="0px" class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username"  prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码区域 -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
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
    data() {
      return {
        // 登录表单数据绑定对象
        loginForm:{
          username:'admin',
          password:'123456'
        },
        // 验证规则
        loginFormRules:{
          // 验证用户名
          username: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
          ],
        }
      };
    },

    created() {},

    methods: {
      // 重置
      resetLoginForm(){
        // console.log(this);
        this.$refs.loginFormRef.resetFields()
      },
      // 登录
      login(){
        this.$refs.loginFormRef.validate(async valid=>{
          // console.log(valid);
         const {data:res}=await this.$http.post('login',this.loginForm)
         
         if(res.meta.status!==200) return this.$message.error("登录失败");
         
         this.$message.success("登录成功");
        //  console.log(res);
         window.sessionStorage.setItem("token",res.data.token)
         this.$router.push("/home")
        })
      }
    },

  }

</script>

<style lang="less" scoped>
  .login_container {
    height: 100%;
    background-color: #409eff;
  }

  .login_box {
    width: 450px;
    height: 300px;
    background-color: #fff;
    position: absolute;
    border-radius: 3px;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    .avatar_box {
      height: 130px;
      width: 130px;
      border-radius: 50%;
      border: 1px solid #eeeeee;
      padding: 10px;
      box-shadow: 0 0 10px #eeeeee;
      position: absolute;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: #fff;

      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #eeeeee;
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
