<!--
 *  商品详情页面
-->
<template>
  <div class="main-box">
    <div>

      <!--左侧的图片-->
      <div class="image-box">
        <img :src="good.imgs" class="image" />
      </div>

      <!--右侧盒子-->
      <div class="detail-box">

        <!--品名与描述-->
        <div>
          <span style="font-size: 22px"
            ><strong>{{ good.name }}</strong></span
          ><br />
        </div>
        <div style="margin-top: 20px">
          <span style="font-size: 17px;">{{
            good.description
          }}</span>
        </div>

        <!--价格盒子-->
        <div class="price-box" v-if="good.discount < 1">
          <dl>
            <div>
              <dt>原价</dt>
              <dd style="text-decoration: line-through"><b>{{ price }}</b>元</dd>
            </div>
            <div>
              <dt>折扣</dt>
              <dd>{{ discount }}</dd>
            </div>
            <div>
              <dt>现价</dt>
              <dd style="color: red; font-size: 25px"><b>{{ realPrice }}</b>元</dd>
            </div>
          </dl>
        </div>
        <div class="price-box" v-if="good.discount === 1">
          <dl>
            <div>
              <dt>价格</dt>
              <dd style="color: red; font-size: 25px">￥ <b>{{ price }}</b></dd>
            </div>
          </dl>
        </div>

        <!--月销量-->
        <div style="margin-top: 20px">
          <span>月销量：</span>
          <span>{{ good.sales }}</span
          ><br />
          <span style="height: 40px" v-if="showStore">库存：{{ store }}</span>
        </div>

        <!--选择规格-->
        <div
          style="margin-top: 15px; height: 50px"
          v-if="standards.length !== 0"
        >
          <el-radio-group
            v-for="(standard, index) in standards"
            v-model="checkedStandard"
            @change="change(standard)"
            :key="index"
          >
            <el-radio-button
              class="standard"
              :label="standard.value"
            ></el-radio-button>
          </el-radio-group>
        </div>

        <!--选择数量-->
        <div style="margin-top: 20px">
          <el-input-number
            v-model="count"
            controls-position="right"
            :min="1"
            :max="store"
          ></el-input-number>
        </div>

        <!--购买按钮组-->
        <div style="margin-top: 30px">
          <el-button type="success" @click="goToOrder">购买</el-button>
          <el-button type="primary" @click="addToCart" icon="el-icon-shopping-cart-1">加入购物车</el-button>
        </div>

      </div>
    </div>

    <div id="message">
      <div class="title">评论</div>
      <div class="wrapper">
        <div class="content">

          <el-input
              type="textarea"
              :rows="3"
              placeholder="评论内容"
              v-model="content"
              clearable>
          </el-input>

          <el-rate v-model="score"></el-rate>
        </div>
        <div class="btn">

          <el-button type="primary" @click="submit()">提交评价</el-button>

        </div>
        <div class="all">
            <el-card class="list"
                @mouseenter="enter(index)"
                @mouseleave="leave(index)"
                v-for="(data,index) in msg" :key="index"
            >
              <p class="content">{{data.content}} </p>
            </el-card>

        </div>
        <div class="pagination">
          <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="pagination.current"
              :page-sizes="[4,6,8,10]"
              :page-size="pagination.size"
              layout="total, sizes, prev, pager, next, jumper"
              :total="pagination.total">
          </el-pagination>
        </div>
      </div>
    </div>
  </div>

</template>

<script>

export default {
  name: "GoodView",
  data() {
    return {
      userId:null,
      baseApi: this.$store.state.baseApi,
      score: 5,
      good: {},
      goodId: Number,
      price: -1,
      isDiscount: false,
      discount: "",
      standards: [],
      checkedStandard: "",
      store: 0,
      showStore: false,
      count: 1,
      title: "",
      content: "",
      pagination: { //分页后的留言列表
        current: 1, //当前页
        total: null, //记录条数
        size: 4 //每页条数
      },
      msg: []
    };
  },
  methods: {
    getPriceRange(standards) {
      let arr = standards.map((item) => {
        return item.price;
      });
      //选择排序
      for (let i = 0; i < arr.length; i++) {
        // 第一次循环，假设第一个值为最小值，后面i++依此类推
        let min = i;
        for (let j = i + 1; j < arr.length; j++) {
          // 将第一个值循环与后面的值比较，如果后面的值比第一个值小，则将索引赋值给min，直到找到最小值
          if (arr[j] < arr[min]) {
            min = j;
          }
        }
        [arr[i], arr[min]] = [arr[min], arr[i]];
      }
      if (arr[0] === arr[arr.length - 1]) {
        return arr[0];
      } else {
        return arr[0] + "元 ~ " + arr[arr.length - 1];
      }
    },
    change(standard) {
      this.showStore = true;
      this.price = standard.price;
      this.store = standard.store;
    },
    goToOrder() {
      if (this.standards.length !== 0) {
        if (this.checkedStandard === "") {
          this.$message.warning("请选择规格");
          return false;
        }
      }
      this.$router.push({
        name: "preOrder",
        query: {
          good: JSON.stringify(this.good),
          realPrice: this.realPrice,
          num: this.count,
          standard: this.checkedStandard,
        },
      });
    },
    addToCart() {
      //未登录，拦截
      if (!localStorage.getItem("user")) {
        this.$router.push("/login");
      }
      if (!this.checkedStandard) {
        this.$message.error("请选择规格");
        return false;
      }
      // 从服务器获取当前用户的id，保证安全
      this.request.get("/userid").then((res) => {
        let userId = res;
        this.userId = res;
        let cart = {
          userId: userId,
          goodId: this.goodId,
          standard: this.checkedStandard,
          count: this.count,
        };
        this.request.post("/api/cart", cart).then((res) => {
          if (res.code === "200") {
            this.$message.success("成功添加购物车");
          }
        });
      });
    },
    getMsg() {
      // 从服务器获取当前用户的id，保证安全
      this.request.get("/userid").then((res) => {
        this.userId = res;
      });

      this.request.get(`/messages/${this.goodId}/${this.pagination.current}/${this.pagination.size}`).then(res => {
        let status = res.code
        if(status == 200) {
          this.msg = res.data.records
          this.pagination = res.data
        }
      })
    },
    //改变当前记录条数
    handleSizeChange(val) {
      this.pagination.size = val
      this.getMsg()
    },
    //改变当前页码，重新发送请求
    handleCurrentChange(val) {
      this.pagination.current = val
      this.getMsg()
    },

    submit() {
      let date = new Date()
      if(false) { //非空判断
        this.$message({
          type: 'error',
          message: '1',
        })
      } else {
        this.request.post("/message", {
            title: this.title,
            goodId: this.goodId,
            content: this.content,
            score:this.score,
            time: date,
            userId: this.userId
          }
        ).then(res => {
          let code = res.data.code
          if(code == 200) {
            this.$message({
              type: "success",
              message: "评论成功"
            })
          }
          this.getMsg()
        })
      }
      this.title = ""
      this.content = ""
      this.getMsg()
    },
    replay(messageId) { //回复留言功能
      this.$prompt('回复留言', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        inputPattern: /^[\s\S]*.*[^\s][\s\S]*$/,
        inputErrorMessage: '回复不能为空'
      }).then(({ value }) => {
        let date = new Date()
        console.log(messageId)
        this.request.post( '/replay',
          {
            replay: value,
            replayTime: date,
            messageId: messageId
          }
        ).then(res => {
          this.getMsg()
        })
        this.$message({
          type: 'success',
          message: '回复成功'
        });
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '取消输入'
        });
      });
    },
    enter(index) {
      this.flag = true
      this.current = index
    },
    leave(index) {
      this.flag = false;
      this.current = index;
    }
  },

  created() {
    this.goodId = this.$route.params.goodId;
    this.getMsg()
    //初始化商品信息
    // this.good = JSON.parse(this.$route.query.good)
    this.request.get("/api/good/" + this.goodId).then((res) => {
      if (res.code === "200") {
        this.good = res.data;
        let discount = this.good.discount;
        if (discount < 1) {
          this.isDiscount = true;
          this.discount = discount * 10 + "折";
        }
      } else {
        this.$router.go(0);
      }
    });
    //从服务器获取商品规格信息
    this.request.get("/api/good/standard/" + this.goodId).then((res) => {
      if (res.code === "200") {
        let standards = JSON.parse(res.data);
        this.standards = standards;
        //默认选择第一个标准
        this.price = this.getPriceRange(standards);
      } else {
        //没有规格
        this.price = this.good.price;
        this.store = this.good.store;
        this.showStore = true;
      }
    });
  },
  mounted() {},
  computed: {
    //折后价，小数点后2位
    realPrice: function () {
      if (this.good.discount < 1) {
        //价格为范围，即不是数字，则返回一个范围
        if (isNaN(this.price)) {
          let down =
            this.price.substring(0, this.price.indexOf("元")) *
            this.good.discount;
          let up =
            this.price.substring(this.price.lastIndexOf(" ")) *
            this.good.discount;
          return down.toFixed(2) + "元 ~ " + up.toFixed(2);
        } else {
          return (this.price * this.good.discount).toFixed(2);
        }
      }
      return this.price;
    },
  },
};
</script>

<style scoped>
.main-box {
  width: 1060px;
  margin: 20px auto;
  padding: 30px;
  background-color: #ffffff;
  overflow: hidden;
}
.image-box {
  height: 420px;
  width: 420px;
  border: #f2f2f2 1px solid;
  text-align: center;
  margin-left: 80px;
  margin-top: 30px;
  display: inline-block;
  overflow: hidden;
}
.image {
  height: 100%;
  width: 350px;
}
.detail-box {
  width: 420px;
  display: inline-block;
  margin-left: 50px;
  overflow: hidden;
}
.price-box {
  background-color: #e9e9e9;
  border-radius: 5px;
  font: 12px/1.5 "Microsoft Yahei", tahoma, arial;
  padding-bottom: 1px;
  padding-top: 1px;
  margin-right: 20px;
  margin-top: 30px;
}
.price-box div {
  line-height: 20px;
  margin-left: 8px;
  margin-bottom: 5px;
}
.price-box dl dt {
  float: left;
  font-size: 14px;
  line-height: 20px;
}
.price-box dl dd {
  font-size: 18px;
  line-height: 20px;
}
.button {
  width: 130px;
  height: 45px;
  background-color: #96e2e0;
  color: #710a0a;
}
.standard {
  height: 30px;
  margin-right: 10px;
}

.pagination {
  display: flex;
  justify-content: center;
}
#message {
  width: 980px;
  margin: 0 auto;
}
.title {
  margin: 20px;
}
.content {
  padding: 20px 0px;
}
#message  {
  .btn {
    padding-bottom: 20px;
  }
  .all {
    .date {
      color: rgb(80, 157, 202);
      line-height: 45px;
      font-size: 13px;
    }
    .list {
      background-color: #eee;
      padding:10px;
      border-radius: 4px;
      margin: 10px 0px;
      position: relative;
      transition: all .3s ease;
      .title {
        color: #5f5f5f;
        margin: 0px;
        font-size: 13px;
        line-height: 30px;
      }
      .content {
        padding: 0px;
      }
      .icon-untitled33 {
        font-size: 13px;
        margin-right: 4px;
      }
      .icon-date {
        font-size: 13px;
        margin-right: 4px;
        color: rgb(80, 157, 202);
      }
      .replay {
        position: absolute;
        right: 30px;
        bottom: 15px;
        color: tomato;
        cursor: pointer;
        transition: all .3s ease;
      }
      .comment {
        margin:-7px 0px;
        padding-bottom: 12px;
        font-size: 13px;
        color: #28b2b4;
        i {
          margin-right: 4px;
        }
      }
    }
  }
}
#message .wrapper {
  background-color: #fff;
  padding: 20px;

}
</style>
