<template>
    <div>
        <el-container>
            <el-header class="homeHeader">
                <div class="title">控制器</div>
                <el-dropdown class="userInfo" @command="commandHandle">
  <span class="el-dropdown-link">
      {{user.name}}<i><img :src="user.userface" alt=""></i>
  </span>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item command="userInfo">个人中心</el-dropdown-item>
                        <el-dropdown-item command="setting">设置</el-dropdown-item>
                        <el-dropdown-item command="logout" divided>注销登录</el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </el-header>
            <el-container>
                <el-aside width="200px">
                    <el-menu router unique-opened>
                        <el-submenu :index="index+''" v-for="(item,index) in routes" v-if="!item.hidden" :key="index">
                            <template slot="title">
                                <i style="margin-right: 5px;color: #2892ea" :class="item.iconCls"/>
                                <span>{{item.name}}</span>
                            </template>
                            <el-menu-item :index="child.path"v-for="(child,indexj) in item.children" :key="indexj">{{child.name}}</el-menu-item>
                        </el-submenu>
                    </el-menu>
                </el-aside>
                <el-main>
                    <el-breadcrumb separator-class="el-icon-arrow-right" v-if="this.$router.currentRoute.path!=='/Home'">
                        <el-breadcrumb-item :to="{ path: '/Home' }">首页</el-breadcrumb-item>
                        <el-breadcrumb-item>{{this.$router.currentRoute.name}}</el-breadcrumb-item>
                    </el-breadcrumb>
                    <div class="homeWelcome" v-if="this.$router.currentRoute.path ==='/Home'">
                        欢迎来到控制台
                    </div>
                    <router-view class="routerView"/>
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
    export default {
        name: "Home",
        data() {
            return {
                user: JSON.parse(window.sessionStorage.getItem("user"))
            }
        },
        computed:{
          routes(){
              return this.$store.state.routes;
          }
        },
        methods: {
            commandHandle(cmd) {
                if (cmd === 'logout') {
                    this.$confirm('此操作将注销用户, 是否继续?', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }).then(() => {
                        this.getRequest("logout");
                        window.sessionStorage.removeItem("user");
                        this.$store.commit('initRoutes', []);
                        this.$router.replace("/");
                    }).catch(() => {
                        this.$message({
                            type: 'info',
                            message: '已取消'
                        });
                    });
                }
            }
        }
    }
</script>

<style scoped>
    .homeWelcome{
        text-align: center;
        font-size: 30px;
        color: #505458;
        padding-top:46px;
    }
    .homeHeader {
        background: #2892ea;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 15px;
        box-sizing: border-box;
    }

    .homeHeader .title {
        font-size: 20px;
        font-family: "微软雅黑";
        color: white;
    }

    .userInfo {
        cursor: pointer;
    }

    .el-dropdown-link img {
        width: 46px;
        height: 46px;
        border-radius: 23px;
        margin-left: 8px;
    }

    .el-dropdown-link {
        display: flex;
        align-items: center;
    }
    .routerView{
        margin-top: 13px;
    }
</style>