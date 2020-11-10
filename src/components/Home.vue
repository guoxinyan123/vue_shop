<template>
      <el-container class="home-container">
        <el-header>
          <div>
            <img src="../assets/heima.png" alt="">
            <span>电商后台管理系统</span>
          </div>
          <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
          <el-aside :width="isCollapse ? '64px' : '200px' ">
            <div class="toggle-button" @click="toggleCollapse">|||</div>
            <el-menu
              :default-active="activePath"
              :router="true"
              :collapse-transition="false"
              :collapse="isCollapse"
              unique-opened
              background-color="#333744"
              text-color="#fff"
              active-text-color="#409EFF">
              <el-submenu :index="item.id + '' " :key="item.id" v-for="item in menuList">
                <template slot="title">
                  <i :class="iconObj[item.id]"></i>
                  <span>{{item.authName}}</span>
                </template>
                  <el-menu-item :index=" '/' + subItem.path" v-for="subItem in item.children" :key="subItem.id"  @click="saveNavState('/' + subItem.path)">
                    <template slot="title">
                      <i class="el-icon-menu"></i>
                      <span>{{subItem.authName}}</span>
                    </template>
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
  data: function () {
    return {
      menuList: [],
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created: function () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout: function () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    getMenuList: async function () {
      var { data: res } = await this.$http.get('menus')
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    toggleCollapse: function () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState: function (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
  .el-header {
    background-color: #373d41;
    color: #fff;
    font-size: 20px;
    padding-left: 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    > div {
      display: flex;
      align-items: center;
      span {
        margin-left: 20px;
      }
    }
  }
  .el-aside {
    background-color: #313541;
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: #e3e6e9;
  }
  .iconfont {
    margin-right: 10px;
  }
  .toggle-button {
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
}
</style>
