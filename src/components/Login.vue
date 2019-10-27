<template>
  <div class="login">
    <el-form ref="form" :rules="rules" :model="form" label-width="80px">
      <img src="../assets/avatar.jpg" class="logo" />
      <el-form-item label="用户名" prop="username">
        <el-input @keyup.enter.native="login" v-model="form.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input @keyup.enter.native="login" type="password" v-model="form.password" placeholder="请输入密码"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="login" class="loginBtn" type="primary">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: ['change', 'blur']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['change', 'blur']
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: ['change', 'blur']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['change', 'blur']
          }
        ]
      }
    }
  },
  methods: {
    reset () {
      this.$refs.form.resetFields()
    },
    async login () {
      const { meta, data } = await this.$axios.post('login', this.form)
      if (meta.status === 200) {
        console.log(meta)
        this.$message({
          type: 'success',
          message: meta.msg,
          duration: 1000
        })
        localStorage.setItem('token', data.token)
        this.$router.push('/index')
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style lang="scss">
.login {
  width: 100%;
  height: 100%;
  background-color: #2d434c;
  overflow: hidden;
  .el-form {
    width: 400px;
    margin: 0 auto;
    margin-top: 200px;
    padding: 20px;
    padding-top: 75px;
    border-radius: 20px;
    background-color: #fff;
    position: relative;
    .logo {
      border-radius: 50%;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      border: 5px solid #fff;
      top: -75px;
    }
    .loginBtn {
      margin-right: 70px;
    }
  }
}
</style>
