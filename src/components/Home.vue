<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <!-- 组件名称就是类名 -->
    <el-header>
      <div>
        <img src="../assets/logo.png" height="50px" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollaps">&lt;&lt;&lt;</div>
        <!-- 侧边栏菜单栏区域 -->
        <el-menu background-color="#333744"
        text-color="#fff"
        active-text-color="#409BFF"
        unique-opened
        :collapse="isCollapse"
        :collapse-transition="false"
        :router="true"
        :default-active="activePath">
        <!-- 也可以default-active="$router.path"> -->
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children"
            :key="subItem.id"
            @click="saveNavState('/'+subItem.path)">
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
      <!-- 右侧内容 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      iconsObj: {
        125: 'el-icon-s-operation',
        103: 'el-icon-s-operation',
        101: 'el-icon-s-operation',
        102: 'el-icon-s-operation',
        145: 'el-icon-s-operation'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(res)
    },
    // 点击按钮，切换折叠和展开
    toggleCollaps () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (activePath) {
      sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding: 10px;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
// .el-menu {
//   width: 200px;
// }
.el-icon-arrow-up {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4A5064;
  font-size: 15px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
