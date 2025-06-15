<!--
 * 前台页面-导航栏
-->
<template>
  <div class="navagation">
    <el-row>

      <el-col :span="3">
        <div style="font-size: 20px; font-weight: bold; text-align: center">
          <a href="/"> <img src="../resource/logo.png" style="width: 100px; margin-right: 0px;"></a>
        </div>
      </el-col>

      <el-col :span="17">
        <el-menu :default-active="activeIndex" class="el-menu-demo" mode="horizontal" router>

          <el-menu-item index="/" class="menu-item">商城首页</el-menu-item>
          <el-menu-item index="/goodList" class="menu-item">商品分类</el-menu-item>
          <el-menu-item index="/cart" class="menu-item">我的购物车</el-menu-item>
          <el-menu-item index="/orderlist" class="menu-item">我的订单</el-menu-item>
          <el-menu-item index="/manage" class="menu-item" v-if="role === 'admin'">后台管理</el-menu-item>

        </el-menu>
      </el-col>

      <el-col :span="4">
        <div style="float: right; margin-right: 60px;">
          <!-- 未登录时显示登录链接 -->
          <a v-if="!loginStatus" @click="$router.push({ path: '/login', query: { to: '/' } })" style="cursor: pointer;">
            登录
          </a>

          <!-- 登录后显示用户信息和下拉菜单 -->
          <el-dropdown v-else>
            <span class="el-dropdown-link">
              <div style="display: inline-block; cursor: pointer;">
                <img v-if="user.avatarUrl != null" :src="user.avatarUrl" class="avatar" />
                {{ user.nickname }}
                <i class="el-icon-arrow-down el-icon--right" style="margin-right: 5px;"></i>
              </div>
            </span>

            <!-- 用户信息下拉菜单 -->
            <el-dropdown-menu slot="dropdown" style="text-align: center;">
              <el-dropdown-item>
                <div @click="$router.push('/person')">个人信息</div>
              </el-dropdown-item>
              <el-dropdown-item>
                <div @click="logout">退出</div>
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </div>
      </el-col>
    </el-row>
  </div>
</template>


<script>
export default {
  name: "Navagation",
  props: {
    user: Object,
    loginStatus: Boolean,
    role: String,
  },
  //初始化数据
  data() {
    return {
      activeIndex: "1",
      activeIndex2: "1",
      baseApi: this.$store.state.baseApi,
    };
  },
  methods: {
    //退出账号
    logout() {
      localStorage.removeItem("user");
      this.$router.go(0);
      this.$message.success("退出成功");
    },
  },
};
</script>
<style>
a {
  text-decoration: none;
}

.navagation {
  width: 100%;
  height: 60px;
  line-height: 60px;
  background-color: #ffffff;
  overflow: hidden;
}

.avatar {
  width: 30px;
  border-radius: 50%;
  position: relative;
  top: 10px;
  right: 5px;
}

.menu-item {
  padding-left: 50px;
  padding-right: 50px;
}
</style>
