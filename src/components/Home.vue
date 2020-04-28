<template>
  <div>
    <el-container class="home_container">
        <!--头部-->
        <el-header>Header
            <el-button type="info" @click="loginout">退出</el-button>
        </el-header>
        <!---->
        <el-container>
            <!--左侧菜单-->
            <el-aside width="200px">Aside</el-aside>
            <!--右侧主体-->
            <el-main>Main</el-main>
        </el-container>
    </el-container>
  </div>
</template>

<script>
export default {
  methods: {
    loginout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
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
}
.el-aside{
    background-color: #333744;
}
.el-main{
    background-color: #eaedf1;
}
</style>
