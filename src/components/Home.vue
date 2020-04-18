<template>
  <el-container class="home-container">
    <!--头部区-->
    <el-header>
      <div>
        <img src="../assets/logo.png" style="height: 50px; width: 50px;" alt=""/>
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>
    </el-header>

    <!--页面主体区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="isCollapse ? '64' : '200'">
        <div class="toggle-button" @click="toggleCollapse">
          <i :class="isCollapse ? 'el-icon-s-unfold' : 'el-icon-s-fold'"></i>
        </div>
        <!--侧边栏菜单区域-->
        <el-menu background-color="#333744" text-color="#fff"
                 active-text-color="#409EFF" :unique-opened="true"
                 :collapse="isCollapse" :collapse-transition="false"
                 :router="true" :default-active="activePath">

          <!--一级菜单-->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!--一级菜单模板-->
            <template slot="title">
              <i :class="item.iconName"></i>
              <span>{{item.authName}}</span>
            </template>

            <!--二级菜单-->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children"
                          :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>

          </el-submenu>
        </el-menu>
      </el-aside>
      <!--内容区域-->
      <el-main>
        <!--路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>

</template>

<script>
  export default {

    data() {
      return {
        //左侧菜单栏数据
        menuList: [
          {
            "id": 1,
            "authName": "权限管理",
            "iconName": "el-icon-setting",
            "children": [
              {
                "id": 11,
                "authName": "权限列表",
                "path": "rights"
              },
              {
                "id": 12,
                "authName": "角色列表",
                "path": "roles"
              }
            ]
          },
          {
            "id": 2,
            "authName": "用户管理",
            "iconName": "el-icon-user",
            "children": [
              {
                "id": 21,
                "authName": "用户列表",
                "path": "users"
              }
            ]
          },
          {
            "id": 3,
            "authName": "个人中心",
            "iconName": "el-icon-user",
            "children": [
              {
                "id": 31,
                "authName": "个人信息",
                "path": "info"
              },
              {
                "id": 32,
                "authName": "修改密码",
                "path": "updataPassword"
              }
            ]
          },
          {
            "id": 4,
            "authName": "阿萨德",
            "iconName": "el-icon-user",
            "children": [
              {
                "id": 41,
                "authName": "阿萨德",
                "path": "info"
              }
            ]
          },
        ],
        //是否折叠，默认不折叠
        isCollapse: false,
        //被激活的链接地址
        activePath: ''
      }
    },

    //页面加载时首先执行方法
    created() {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    },

    methods: {

      loginOut () {
        //清空token,转到登录页
        window.sessionStorage.clear()
        this.$router.push('/login')
      },

      //获取左侧菜单栏菜单
      async getMenuList() {
        const {data: res} = this.$http.get('/menus')
        if(res.code != '200') return this.$message.error(res.resulet)
        this.menuList = res.data
        console.log(res)
      },

      //点击按钮切换菜单的折叠与展开
      toggleCollapse() {
        this.isCollapse = ! this.isCollapse
      },
      //保存链接的激活状态
      saveNavState(activePath) {
        window.sessionStorage.setItem('activePath',activePath)
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
    background-color: #333744;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
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
    background-color: white;
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
    cursor: pointer;
  }
</style>
