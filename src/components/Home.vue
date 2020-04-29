<template>
    <el-container class="home_container">
        <!--头部-->
        <el-header>
          <div>
            <img src="../assets/heima.png" alt="">
            <span>电商后台管理系统</span>
          </div>
          <el-button type="info" @click="loginout">退出</el-button>
        </el-header>
        <!---->
        <el-container>
            <!--侧边栏-->
            <el-aside :width="iscollapse ? '64px':'200px'">
              <!--菜单折叠与展开按钮-->
              <div class="toggle-button" @click="toggleCollapse">|||</div>
              <!--菜单区域-->
               <el-menu
                  background-color="#333744"
                  text-color="#fff"
                  active-text-color="#409eff"
                  :unique-opened="true"
                  :collapse="iscollapse"
                  :collapse-transition="false"
                  :router="true"
                  :default-active="activePath">
                  <el-submenu :index="'/'+item.path" v-for="item in menuList" :key="item.id">
                    <!--一级菜单-->
                    <template slot="title">
                        <i :class="iconObj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <!--二级菜单-->
                    <el-menu-item @click="saveActivePath('/'+item2.path)" :index="'/'+item2.path" v-for="item2 in item.children" :key="item2.id">
                      <template slot="title">
                          <i class="el-icon-menu"></i>
                          <span>{{item2.authName}}</span>
                      </template>
                    </el-menu-item>
                  </el-submenu>
               </el-menu>
            </el-aside>
            <!--右侧主体-->
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
      menuList: [],
      iconObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      iscollapse: false,
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    loginout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取菜单列表失败')
      console.log(res.data)
      this.menuList = res.data
    },
    toggleCollapse () {
      this.iscollapse = !this.iscollapse
    },
    saveActivePath (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home_container{
  height: 100%;
}
.el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #fff;
    font-size: 20px;
    div{
      display: flex;
      align-items: center;
      span{
        margin-left: 15px;
      }
    }
}
.el-aside{
    background-color: #333744;
}
.el-main{
    background-color: #eaedf1;
}
.iconfont{
  margin-right: 15px;
}
.el-menu{
  border-right: none;
}
.toggle-button{
  color: #fff;
  text-align: center;
  background-color: #4A5064;
  font-size: 10px;
  letter-spacing: 0.3em;
  line-height: 24px;
  cursor: pointer;
}
</style>
