<!--
 * 购物车内容项
-->
<template>
  <div>
    <div class="header" style="padding-left: 25px;">
      <span style="line-height: 40px">加入购物车时间：{{ cart.createTime }}</span>
    </div>
    <div class="body">

     <!--商品图片-->
      <div style="display: inline-block;margin-right: 20px">
        <router-link :to="'goodview/'+cart.goodId">
          <img :src="cart.img" style="width: 100px;height:100px">
        </router-link>
      </div>

      <!-- 商品信息-->
      <div style="display: inline-block;line-height: 40px" >
        <table>
          <tr>
            <th>商品</th>
            <th>规格</th>
            <th>价格</th>
            <th>数量</th>
            <th>总价</th>
            <th>操作</th>
          </tr>

          <tr>
            <a :href="'goodview/'+cart.goodId">
              <td>{{ cart.goodName }}</td>
            </a>
            <td>{{cart.standard}}</td>
            <td>{{realPrice}}</td>
            <td>
              <el-button size="mini" @click="countChangeFlag=true" v-if="!countChangeFlag">
                {{cart.count}}
              </el-button>
              <el-input-number v-model="cart.count" :min="1" :max="cart.store" v-if="countChangeFlag" style="width: 120px" ></el-input-number>
              </td>
            <td>{{totalPrice}}</td>
            <td>
              <el-button size="mini" type="success" @click="pay">支付</el-button>
              <el-popconfirm
                  @confirm="del"
                  title="确定删除？"
              >
                <el-button size="mini" type="danger" icon="el-icon-delete" slot="reference" style="margin-left: 10px"></el-button>
              </el-popconfirm>
            </td>

          </tr>

        </table>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: "CartItem",
  props:{
    cart: Object,
    countChangeFlag: false,
  },
  created() {

  },
  data(){
    return{

      baseApi: this.$store.state.baseApi,
    }
  },
  computed:{
    totalPrice:function () {
      return (this.realPrice * this.cart.count).toFixed(2)
    },
    realPrice: function (){
      return (this.cart.price * this.cart.discount)
    }
  },
  methods:{
    //从购物车移除
    del(id){
      this.request.delete("/api/cart/"+this.cart.id).then(res=>{
        if(res.code==='200'){
          this.$message.success("删除成功")
          this.$emit('delete',this.cart.id)
        }
      })
    },
    //跳转到支付页面
    pay(){
      let good = {id: this.cart.goodId,name: this.cart.goodName,imgs: this.cart.img,discount: this.cart.discount}
      this.$router.push({name: 'preOrder',query: {good: JSON.stringify(good), realPrice: this.realPrice, num: this.cart.count, standard: this.cart.standard, cartId: this.cart.id}})
    },
  }
}
</script>

<style scoped>
.header{
  background-color: #e1d8cd;
  height: 40px;
}
.body{
  background-color: white;
  padding: 20px;
}
th,td{

  width: 120px;
  text-align: center;
}
th{
  font-size: 15px;
  color: #3472a6;
  font-weight: normal;
}
</style>
