<template>
  <el-container class="home-container">
    <!-- 头部 start -->
    <el-header>
      <!-- logo start -->
      <div>
        <img src="../assets/heima.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <!-- logo end -->
      <!-- 退出按钮 start-->
      <el-button type="info" @click="logout">退出</el-button>
      <!-- 退出按钮 end-->
    </el-header>
    <!-- 头部 end -->
    <el-container>
      <!-- 侧边栏 start -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 折叠按钮 -->
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单 start -->
        <el-menu
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409eff"
          unique-opened
          :collapse="isCollapse" 
          :collapse-transition="false"
          router
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="index + ''"
            v-for="(item, index) in menulist"
            :key="index"
          >
            <template slot="title">
              <i :class="icons[index]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              v-for="(subitem, index) in item.children"
              :key="index"
              :index=" '/' + subitem.path"
              @click="activeMenu(subitem.path)"
            >
              <template slot="title">
                <i class="el-icon-menu activeMenu" :class="{activeMenuColor : subitem.path == currentPath  }"></i>
                <span :class="{activeMenuColor : subitem.path == currentPath  }">{{ subitem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
        <!-- 侧边栏菜单 end -->
      </el-aside>
      <!-- 侧边栏 end -->
      <!-- 主体部分 start -->
      <el-main>
        <router-view></router-view>
      </el-main>
      <!-- 主体部分 end -->
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      menulist: [],
      icons: {
        0: 'iconfont icon-user',
        1: 'iconfont icon-tijikongjian',
        2: 'iconfont icon-shangpin',
        3: 'iconfont icon-danju',
        4: 'iconfont icon-baobiao',
      },
      isCollapse: false,
      currentPath: '',
    }
  },
  created() {
    //调用获取菜单数据函数
    this.getMenuList()
  },
  methods: {
    //退出功能
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    //获取菜单数据方法
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    //折叠菜单栏
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    //点击二级菜单变色
    activeMenu(path) {
      this.currentPath = path
    }
  },
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
  align-items: center;
  padding-left: 0;
  > div {
    display: flex;
    align-items: center;
    color: #fff;
    font-size: 20px;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
    .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    //三条竖线的间距
    letter-spacing: 0.2em;
    cursor: pointer;
  }
   .el-menu {
    border-right: 0;
  }
}
.el-main {
  background-color: #eaedf1;
}

.iconfont {
  margin-right: 10px;
}
.activeMenuColor {
  color:#409eff
}
</style>