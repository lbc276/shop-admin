<template>
  <el-container class="index">
    <el-header>
      <div class="logo">
        <img src="../assets/logo2.png" />
      </div>
      <div class="title">
        <h1>电商后台管理系统</h1>
      </div>
      <div class="logout">
        欢迎光临~
        <a href="javascript:;" @click="logout">退出</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <!--
          el-menu 整个大的导航组件
            default-active="1-1" 可以控制默认高亮的导航
          el-submenu 子菜单
          el-menu-item 展开的内容项
        -->
        <el-menu
          router
          unique-opened
          :default-active="defaultActive"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
        >
          <el-submenu :index="menu.path" v-for="menu in menuList" :key="menu.id">
            <template v-slot:title>
              <i class="el-icon-location"></i>
              <span>{{menu.authName}}</span>
            </template>
            <el-menu-item v-for="item in menu.children" :key="item.id" :index="item.path">
              <i class="el-icon-menu"></i>
              <span slot="title">{{item.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  computed: {
    defaultActive () {
      return this.$route.path.slice(1)
    }
  },
  async created () {
    const { meta, data } = await this.$axios.get('menus')
    if (meta.status === 200) {
      this.menuList = data
    }
  },
  data () {
    return {
      menuList: []
    }
  },
  methods: {
    logout () {
      this.$confirm('确定要退出本系统吗?', '温馨提示', {
        type: 'warning'
      }).then(() => {
        localStorage.removeItem('token')
        this.$router.push('/login')
      }).catch(e => {
        console.log(e)
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.index {
  height: 100%;
  .el-header {
    background-color: #d8d8d8;
    display: flex;
    .logo {
      width: 200px;
      padding: 10px;
      img {
        height: 40px;
      }
    }
    .title {
      flex: 1;
      h1 {
        text-align: center;
        line-height: 60px;
      }
    }
    .logout {
      width: 200px;
      text-align: right;
      line-height: 60px;
      font-weight: 700;
      a {
        color: orange;
      }
    }
  }
  .el-aside {
    background-color: #545c64;
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: #ecf0f1;
  }
}
</style>
