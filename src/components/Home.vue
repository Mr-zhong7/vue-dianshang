<template>
  <el-container>
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <!-- 展开和关闭 -->
        <div class="toggle-button" @click=toggleCollapse>|||</div>
        <!-- 侧边栏菜单区域 -->
        <el-menu background-color="#313743" text-color="#fff" active-text-color="#449dfd" unique-opened
          :collapse-transition="false" :collapse="isCollapse" router :default-active=activePath>
          <!-- 一级菜单 -->
          <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
            @click="saveNavState('/'+subItem.path)"
            >
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容主体 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data() {
      return {
        // 右侧菜单数
        menulist: [],
        // 图标
        iconsObj: {
          '125': 'iconfont icon-user',
          '103': 'iconfont icon-tijikongjian',
          '101': 'iconfont icon-shangpin',
          '102': 'iconfont icon-danju',
          '145': 'iconfont icon-baobiao'
        },
        // 点击按钮，切换菜单展开与折叠
        isCollapse: false,
        // 保存链接激活地址
        activePath:''
      };
    },

    created() {
      //获取左侧所有菜单数据
      this.getMenuList()
      // 激活高亮状态
      this.activePath=window.sessionStorage.getItem('activePath')
    },

    methods: {
      // 退出登录
      loginOut() {
        window.sessionStorage.clear()
        this.$router.push("/login")
      },
      // 获取左侧所有菜单数据
      async getMenuList() {
        const {
          data: res
        } = await this.$http.get('menus')
        // console.log(res);
        if (res.meta.status !== 200) return this.$message.error('res.meta.msg')
        this.menulist = res.data
      },
      // 点击按钮，切换菜单展开与折叠
      toggleCollapse() {
        this.isCollapse = !this.isCollapse
      },
      //保存链接激活状态
      saveNavState(activePath){
       window.sessionStorage.setItem("activePath",activePath)
        this.activePath=activePath
      }
    },

  }

</script>

<style lang="less" scoped>
  .el-container {
    height: 100%;
  }

  .el-header {
    background-color: #363d40;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #ffffff;
    font-size: 20px;

    >div {
      display: flex;
      align-items: center;
    }

    span {
      margin-left: 15px;
    }
  }

  .el-aside {
    background-color: #313743;

    .el-menu {
      border-right: none;
    }
  }

  .el-main {
    background-color: #e9eef3;
  }

  .iconfont {
    margin-right: 10px;
  }

  .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    color: #ffffff;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }

</style>
