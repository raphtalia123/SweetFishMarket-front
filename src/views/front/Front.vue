<!--
 * 前台首页
-->
<template>
  <el-container style="height: 100%;width:100%;">

    <el-header style="background-color: white">
      <Navagation :user="user" :role="role" :login-status="loginStatus"></Navagation>
    </el-header>

    <el-main style="
    background-image: url('/images/background.png'); 
    background-size: contain; 
    background-repeat: repeat;
    background-position: center;
    ">
      <router-view />
    </el-main>

  </el-container>
</template>

<script>

import Navagation from "@/components/Navagation";
import request from "@/utils/request";

export default {
  name: "Front",
  data() {
    return {
      user: {},
      role: 'user',
      loginStatus: false,
    }
  },
  methods: {
    getUser() {
      let username = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).username : ""
      if (username) {
        // 从后台获取User数据
        this.request.get("/userinfo/" + username).then(res => {
          // 重新赋值后台的最新User数据
          this.user = res.data
        })
      }

    },
  },

  components: {
    Navagation,
  },
  created() {
    if (localStorage.getItem("user")) {
      request.post("http://localhost:8888/role").then(res => {
        if (res.code === '200') {
          this.role = res.data;
          if (localStorage.getItem("user")) {
            this.user = JSON.parse(localStorage.getItem("user"));
            this.loginStatus = true;
          }
        } else {
          this.user = { nickname: '您未登录', avatarUrl: null };
          localStorage.removeItem('user')
          this.loginStatus = false;
        }
      })
    } else {
      this.user = { nickname: '您未登录', avatarUrl: null };
      this.loginStatus = false;
    }

  }
}
</script>

<style scoped>
@import "../../resource/css/search.css";

.image {
  width: 100%;
  display: block;
}
</style>
