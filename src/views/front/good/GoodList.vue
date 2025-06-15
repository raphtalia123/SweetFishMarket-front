<!--
 * 商品列表
-->
<template>
  <div>
    <search @search="handleSearch"></search>

    <div class="main-box">
      <!-- 商品分类标题区域 -->
      <div class="category-header">
        <h2 class="page-title">
          <span class="title-icon">🛍️</span>
          商品分类
          <span class="title-subtitle">精选好物，品质保证</span>
        </h2>
      </div>

      <!-- 商品分类菜单 -->
      <div class="category-menu-container">
        <el-row :gutter="20">
          <el-col v-for="(item, index) in icons" :key="index" :span="6">
            <div class="category-card">
              <div class="category-icon">
                <i class="iconfont" v-html="item.value"></i>
              </div>
              <div class="category-links">
                <span v-for="(category, index2) in item.categories" :key="index2">
                  <a href="#"
                    @click.prevent="load(category.id)"
                    :class="{
                      'category-link active': categoryId == category.id,
                      'category-link': categoryId != category.id,
                    }"
                  >
                    {{ category.name }}
                  </a>
                  <span v-if="index2 != item.categories.length - 1" class="separator"> / </span>
                </span>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>

      <!-- 商品展示区域 -->
      <div class="goods-section">
        <div class="section-header">
          <h3 class="section-title">
            <span class="title-icon">⭐</span>
            商品列表
            <span class="goods-count">（共 {{ total }} 件商品）</span>
          </h3>
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
                      <span class="good-price">¥{{ good.price }}</span>
                      <span class="good-tag">精选</span>
                    </div>
                  </div>
                </el-card>
              </router-link>
            </el-col>
          </el-row>
        </div>

        <!-- 分页控件 -->
        <div class="pagination-container">
          <el-pagination
            background
            :hide-on-single-page="false"
            :current-page="currentPage"
            :page-size="pageSize"
            layout="total, prev, pager, next"
            :total="total"
            @current-change="handleCurrentPage"
          >
          </el-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import search from "../../../components/Search";
export default {
  name: "GoodList",
  data() {
    return {
      //分类icon,每个icon包含id、value、categories对象数组.category:id,name
      icons: [],
      total: 0,
      pageSize: 9,
      currentPage: 1,
      //选择的分类
      categoryId: Number,
      //搜索的内容
      searchText: "",
      good: [],
      baseApi: this.$store.state.baseApi,
    };
  },
  components: {
    search,
  },
  created() {
    //二者一般不同时存在
    this.searchText = this.$route.query.searchText;
    this.categoryId = this.$route.query.categoryId;

    this.loadCategories();
    this.load();
  },
  methods: {
    loadCategories() {
      this.request.get("/api/icon").then((res) => {
        if (res.code === "200") {
          this.icons = res.data;
        }
      });
    },
    handleCurrentPage(currentPage) {
      this.currentPage = currentPage;
      this.load();
    },
    handleSearch(text) {
      this.searchText = text;
      this.load();
    },
    load(categoryId) {
      if (categoryId != undefined) {
        this.categoryId = categoryId;

        this.$router.push({
          path: "/goodlist",
          query: { categoryId: this.categoryId },
        });
      }
      this.request
        .get("/api/good/page", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize,
            searchText: this.searchText,
            categoryId: this.categoryId,
          },
        })
        .then((res) => {
          if (res.code === "200") {
            this.total = res.data.total;
            this.good = res.data.records;
          }
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

/* 页面标题区域 */
.category-header {
  text-align: center;
  margin-bottom: 30px;
}

.page-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 28px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.title-icon {
  font-size: 32px;
}

.title-subtitle {
  font-size: 14px;
  color: #999;
  font-weight: normal;
  margin-left: 10px;
}

/* 分类菜单容器 */
.category-menu-container {
  margin-bottom: 40px;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 15px;
}

.category-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s;
  margin-bottom: 15px;
  height: 120px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.category-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.category-icon {
  margin-bottom: 15px;
}

.category-icon .iconfont {
  font-size: 32px;
  color: #667eea;
}

.category-links {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 5px;
}

.category-link {
  color: #666;
  text-decoration: none;
  font-size: 14px;
  padding: 4px 8px;
  border-radius: 12px;
  transition: all 0.3s;
  font-weight: 500;
}

.category-link:hover {
  color: #3186cb;
  background-color: #e6f7ff;
}

.category-link.active {
  color: white;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.separator {
  color: #ccc;
  margin: 0 2px;
}

/* 商品展示区域 */
.goods-section {
  margin-top: 30px;
}

.section-header {
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 22px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

.goods-count {
  font-size: 14px;
  color: #999;
  font-weight: normal;
  margin-left: 10px;
}

.goods-grid {
  margin-bottom: 30px;
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

/* 分页容器 */
.pagination-container {
  text-align: center;
  padding: 20px 0;
  background: #f8f9fa;
  border-radius: 12px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .category-card {
    height: auto;
    min-height: 100px;
  }
}

@media (max-width: 768px) {
  .page-title {
    font-size: 24px;
    flex-direction: column;
    gap: 5px;
  }
  
  .title-icon {
    font-size: 28px;
  }
  
  .category-card {
    height: auto;
    padding: 15px;
  }
  
  .category-icon .iconfont {
    font-size: 24px;
  }
  
  .section-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
}
</style>