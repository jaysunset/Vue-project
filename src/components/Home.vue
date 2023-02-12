<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div class="top">
        <img src=".././assets/images/maer.jpg" alt="" />
        <h2>电商后台管理系统</h2>
      </div>
      <el-button type="info" @click="exit">退出</el-button></el-header
    >
    <!-- 主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单区域 -->
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409BFF"
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + v.path"
              v-for="v in item.children"
              :key="v.id"
              @click="saveNavState('/' + v.path)"
            >
              <!-- 二级菜单的模板区域 -->
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{ v.authName }}</span>
              </template></el-menu-item
            >
          </el-submenu>
        </el-menu></el-aside
      >
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
      // 左侧菜单数据
      menulist: [],
      iconObj: {
        "125": 'el-icon-s-custom',
        "103": 'el-icon-s-cooperation',
        "101": 'el-icon-s-shop',
        "102": 'el-icon-s-order',
        "145": 'el-icon-s-marketing'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ""


    }
  },
  created() {//生命周期钩子 创建完毕时调用
    this.getMenuList(),
      this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    exit() {
      window.sessionStorage.clear()//清空token
      this.$router.push('/login')//重定向到 登录页
    },
    // 获取所有表单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      this.menulist = res.data
      if (res.data.status !== 200) return this.$message.error(res.mate.msg)
      // 点击按钮，切换菜单的折叠与展开


    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem("activePath", activePath)
      this.activePath = activePath
    }
  },
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

.el-header {
  background-color: #373d41;
  display: flex;
}
.top {
  flex: 15;
  display: flex;
}
.el-header img {
  width: 50px;
  height: 50px;
  margin: 5px 10px;
  margin-left: -20px;
  border-radius: 50%;
}
.el-header h2 {
  margin: 10px 10px;
}
.el-header > .el-button {
  flex: 1;
  margin: 10px;
}
.el-aside {
  background-color: #333744;
  transition: 0.5s;
}
.el-aside > .el-menu {
  border-right: 0;
}
.el-main {
  background-color: #eaedf1;
}
.home-container {
  width: 100%;
  height: 100%;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  text-align: center;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  letter-spacing: 0.2em; /* 文本之间的距离 */
  cursor: pointer;
}
</style>