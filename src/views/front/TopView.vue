<!--
 * 前台首页 相关内容
-->
<template>
  <div>
    <search @search="handleSearch"></search>
    <div class="main-box">
      <!-- 主要内容区域 -->
      <div class="content-section">
        <!-- 左侧商品分类 -->
        <div class="good-menu">
          <h3 class="menu-title">商品分类</h3>
          <ul v-for="(item, index) in icons" :key="index">
            <li>
              <i class="iconfont" v-html="item.value"></i>
              <span v-for="(category, index2) in item.categories" :key="index2">
                <router-link
                  :to="{
                    path: '/goodlist',
                    query: { categoryId: category.id },
                  }"
                >
                  <a href="/person"><span class="category-link">{{ category.name }}</span></a>
                </router-link>
                <span v-if="index2 != item.categories.length - 1"> / </span>
              </span>
            </li>
          </ul>
        </div>

        <!-- 中间轮播图 -->
        <div class="carousel-section">
          <el-carousel height="370px" class="main-carousel">
            <el-carousel-item v-for="carousel in carousels" :key="carousel.id">
              <router-link :to="'/goodview/' + carousel.goodId">
                <img class="carousel-img" :src="carousel.img" />
              </router-link>
            </el-carousel-item>
          </el-carousel>
        </div>

        <!-- 右侧信息卡片 -->
        <div class="side-info">
          <!-- 热门推荐小卡片 -->
          <div class="info-card">
            <h4 class="card-title">🔥 今日热销</h4>
            <div class="hot-items" v-if="good.length > 0">
              <div 
                class="hot-item" 
                v-for="(item, index) in good.slice(0, 3)" 
                :key="item.id"
              >
                <router-link :to="'goodview/' + item.id">
                  <img :src="item.imgs" class="hot-item-img" />
                  <div class="hot-item-info">
                    <p class="hot-item-name">{{ item.name }}</p>
                    <span class="hot-item-price">￥{{ item.price }}</span>
                  </div>
                </router-link>
              </div>
            </div>
          </div>

        </div>
      </div>

      <!-- 推荐商品区域 -->
      <div class="recommend-section">
        <div class="section-header">
          <h2 class="section-title">
            <span class="title-icon">⭐</span>
            推荐商品
            <span class="title-subtitle">为您精心挑选</span>
          </h2>
        </div>
        
        <div class="goods-grid">
          <el-row :gutter="20">
            <el-col
              :span="6"
              v-for="good in good"
              :key="good.id"
              style="margin-bottom: 20px"
            >
              <router-link :to="'goodview/' + good.id">
                <el-card class="good-card" :body-style="{ padding: '0px' }">
                  <div class="good-img-container">
                    <img :src="good.imgs" class="good-img" />
                    <div class="good-overlay">
                      <span class="view-detail">查看详情</span>
                    </div>
                  </div>
                  <div class="good-info">
                    <h4 class="good-name">{{ good.name }}</h4>
                    <div class="good-price-container">
                      <span class="good-price">￥{{ good.price }}</span>
                      <span class="good-tag">热销</span>
                    </div>
                  </div>
                </el-card>
              </router-link>
            </el-col>
          </el-row>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import search from "../../components/Search";
export default {
  name: "TopView",
  data() {
    return {
      userId: null,
      carousels: [],
      good: [],
      baseApi: this.$store.state.baseApi,
      icons: [],
      searchText: "",
    };
  },
  components: {
    search,
  },
  created() {
    if (localStorage.getItem("user")) {
      let user = JSON.parse(localStorage.getItem("user"));
      this.userId = user.id;
    } else {
      this.userId = 0;
    }
    this.request.get(`/api/good/all?userId=${this.userId}`).then((res) => {
      if (res.code === "200") {
        this.good = res.data;
      } else {
        this.$message.error(res.msg);
      }
    });
    this.request.get("/api/icon").then((res) => {
      if (res.code === "200") {
        this.icons = res.data;
        if (this.icons.length > 6) {
          this.icons = this.icons.slice(0, 6);
        }
      }
    });
    this.request.get("/api/carousel").then((res) => {
      if (res.code === "200") {
        this.carousels = res.data;
      }
    });
  },
  methods: {
    handleSearch(text) {
      this.searchText = text;
      this.$router.push({
        path: "/goodList",
        query: { searchText: this.searchText },
      });
    },
  },
};
</script>

<style scoped>
.main-box {
  background-color: white;
  border: white 2px solid;
  border-radius: 20px;
  padding: 30px;
  margin: 5px auto;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* 主要内容区域布局 */
.content-section {
  display: flex;
  gap: 20px;
  margin-bottom: 40px;
}

/* 左侧商品分类 */
.good-menu {
  width: 280px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 15px;
  padding: 20px;
  height: 370px;
  overflow-y: auto;
}

.menu-title {
  color: white;
  font-size: 18px;
  font-weight: bold;
  margin: 0 0 20px 0;
  text-align: center;
  border-bottom: 2px solid rgba(255,255,255,0.3);
  padding-bottom: 10px;
}

.good-menu ul {
  padding: 0;
  margin: 0;
}

.good-menu li {
  list-style: none;
  margin-bottom: 20px;
  padding: 8px 0;
  border-bottom: 1px solid rgba(255,255,255,0.1);
}

.good-menu li:last-child {
  border-bottom: none;
}

.good-menu .iconfont {
  color: #fff;
  margin-right: 10px;
  font-size: 16px;
}

.category-link {
  color: rgba(255,255,255,0.9) !important;
  font-size: 14px !important;
  text-decoration: none;
  transition: color 0.3s;
}

.category-link:hover {
  color: #00b7ff !important;
}

/* 轮播图区域 */
.carousel-section {
  flex: 1;
}

.main-carousel {
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.carousel-img {
  height: 370px;
  width: 100%;
  object-fit: cover;
}

/* 右侧信息卡片 */
.side-info {
  width: 260px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.info-card {
  background: white;
  border-radius: 12px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #f0f0f0;
}

.card-title {
  margin: 0 0 12px 0;
  font-size: 14px;
  font-weight: bold;
  color: #333;
  border-bottom: 2px solid #e6f7ff;
  padding-bottom: 8px;
}

/* 热门推荐样式 */
.hot-items {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.hot-item {
  display: flex;
  align-items: center;
  padding: 6px;
  border-radius: 8px;
  transition: background-color 0.3s;
}

.hot-item:hover {
  background-color: #f8f9fa;
}

.hot-item-img {
  width: 40px;
  height: 40px;
  object-fit: cover;
  border-radius: 6px;
  margin-right: 10px;
}

.hot-item-info {
  flex: 1;
}

.hot-item-name {
  font-size: 12px;
  color: #333;
  margin: 0 0 4px 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.hot-item-price {
  font-size: 12px;
  color: #e75c09;
  font-weight: bold;
}

/* 活动公告样式 */
.notice-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.notice-item {
  display: flex;
  align-items: center;
  font-size: 12px;
  color: #666;
}

.notice-dot {
  width: 6px;
  height: 6px;
  background-color: #52c41a;
  border-radius: 50%;
  margin-right: 8px;
}

.notice-text {
  flex: 1;
}

/* 服务保障样式 */
.service-list {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.service-icon {
  margin-right: 6px;
}

/* 推荐商品区域 */
.recommend-section {
  margin-top: 40px;
}

.section-header {
  margin-bottom: 25px;
  text-align: center;
}

.section-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.title-icon {
  font-size: 28px;
}

.title-subtitle {
  font-size: 14px;
  color: #999;
  font-weight: normal;
}

.goods-grid {
  margin-top: 20px;
}

/* 商品卡片样式 */
.good-card {
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.3s;
  border: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.good-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.good-img-container {
  position: relative;
  overflow: hidden;
}

.good-img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  transition: transform 0.3s;
}

.good-card:hover .good-img {
  transform: scale(1.05);
}

.good-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s;
}

.good-card:hover .good-overlay {
  opacity: 1;
}

.view-detail {
  color: white;
  font-size: 14px;
  padding: 8px 16px;
  border: 2px solid white;
  border-radius: 20px;
  transition: all 0.3s;
}

.view-detail:hover {
  background-color: white;
  color: #333;
}

.good-info {
  padding: 15px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.good-name {
  font-size: 16px;
  color: white;
  margin: 0 0 10px 0;
  font-weight: bold;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.good-price-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.good-price {
  color: #ffeb3b;
  font-size: 18px;
  font-weight: bold;
}

.good-tag {
  background-color: #ff4757;
  color: white;
  font-size: 10px;
  padding: 2px 6px;
  border-radius: 8px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .content-section {
    flex-direction: column;
  }
  
  .good-menu,
  .side-info {
    width: 100%;
  }
  
  .good-menu {
    height: auto;
  }
  
  .side-info {
    flex-direction: row;
    gap: 20px;
  }
  
  .info-card {
    flex: 1;
  }
}

@media (max-width: 768px) {
  .side-info {
    flex-direction: column;
  }
  
  .service-list {
    grid-template-columns: 1fr;
  }
}
</style>