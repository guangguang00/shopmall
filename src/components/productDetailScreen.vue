<template>
  <!--产品详情筛选-->
  <div class="productDetailScreen">
    <headBar title="选择商品"></headBar>
    <div class="myShopOrder">
      <div class="productOrderList">
        <img :src="'http://tianyi.zhongkakeji.com/media/' + detail.image" alt="" class="proLeft">
        <div class="proCenter">
          <p class="name">{{detail.title}}</p>
        </div>
        <div class="proRight">
          <p class="allPrice">￥{{totalPrice}}</p>
          <p class="price">￥{{detail.price}}</p>
        </div>
      </div>
    </div>

    <div class="chooseDesc">
      <div class="chooseMain">
        <h2 class="name">颜色选择</h2>
        <span @click="tabChangeColor(item,index)" class="chooseOne" :class="{'active': colorIndex === index}" v-for="(item, index) in detail.color_info" :key="index">
          <i :style="{'backgroundColor':item.value}"></i>
          <em>{{item.title}}</em>
        </span>
      </div>
      <div class="chooseMain">
        <h2 class="name">联网方式选择</h2>
        <span @click="tabChangeNetwork(item,index)" class="chooseOne" :class="{'active': networkIndex === index}" v-for="(item, index) in detail.mode_info" :key="index">
          <em>{{item.title}}</em>
        </span>
      </div>
      <div class="chooseMain">
        <h2 class="name">门厚度选择</h2>
        <span @click="tabChangeDoor(item,index)" class="chooseOne" :class="{'active': doorIndex === index}" v-for="(item, index ) in detail.value_info" :key="index">
          <em>{{item.value}}</em>
        </span>
      </div>
      <div class="chooseMain">
        <h2 class="name">选择数量</h2>
        <div class="chooseCountNum">
          <span class="minMax minleft" @click='minus()' :class="{'addActive':isaddActive === true}">-</span>
          <input type="text" v-model='valueTextNum' class="valueText" readonly>
          <span class="minMax maxright" @click='add()'>+</span>
        </div>
      </div>
    </div>

    <div class="footerBtn" @click="judgeLoginShow">
      <van-row>
        <van-col span="24">
          <van-button bottom-action>确定</van-button>
        </van-col>
      </van-row>
    </div>
    <!--如果没有登录要提示登录框-->
    <phoneLogin v-show="phoneLoginShow" @login="loginFn" @close="closeLogin"></phoneLogin>
    <!--加载-->
    <van-loading v-if="LoadingOk" type="spinner" color="black" class="loadPosition" />
  </div>
</template>

<script>
  import headBar from './headBar'
  import phoneLogin from './phoneLogin'
  import { UserInfo } from 'common/js/common'
  import { productDetail } from 'api/product'
  export default {
    components: {
      headBar,
      phoneLogin
    },
    props: {
      data: {
        type: Object,
        default: {}
      }
    },
    data() {
      return {
        detail: {},
        LoadingOk: false,
        result: 0,
        valueTextNum: 1,
        isActive: true,
        phoneLoginShow: false,//手机登录框

        colorIndex: -1,
        networkIndex: -1,
        doorIndex: -1,

        orderData:{},

        userInfo:'',


      }
    },
    mounted() {
      this.detail = this.data;
      this.userInfo = UserInfo();
    },
    computed: {//计算属性
      totalPrice() {//计算总价
        let total = 0;
        total += this.valueTextNum * this.detail.price
        return total;
      },
      isaddActive() {//计算购物车数量小于等于1，变灰色不能点
        if(this.valueTextNum <= 1) {
          return true;
        } else {
          return false;
        }
      }
    },
    methods: {
      minus() {//减
        if(this.valueTextNum <= 1) {
          return;
        }
        this.valueTextNum--
      },
      add() {//加
        this.valueTextNum++
      },
      tabChangeColor(item,index) {
        this.colorIndex = index
        this.orderData.color_id = item.color_id;
        this.orderData.colorTitle  = item.title;

      },
      tabChangeNetwork(item,index) {
        this.networkIndex = index
        this.orderData.netWorkTitle  = item.title;
        this.orderData.mode_id =item.mode_id;
      },
      tabChangeDoor(item,index) {
        this.doorIndex = index
        this.orderData.value = item.value;
        this.orderData.value_id = item.value_id;
      },
      //如果没有在个人中心登录的话，点击确定弹出登录框
      judgeLoginShow() {
        //如果没有选择，在页面跳转之前弹框提示
        if(this.colorIndex === -1){
          this.$toast("请选择商品颜色");
          return;
        }
        if(this.networkIndex === -1){
          this.$toast("请选择商品联网方式选择")
          return;

        }
        if(this.doorIndex === -1){
          this.$toast("请选择商品门厚度")
          return;

        }
        if(this.userInfo === '') {
          this.phoneLoginShow = true;
        } else{
          // this.$toast("你以登录成功")
          //如果登陆成功就返回详情页，并把选中的商品内容传给详情页
          //this.$router.push({name: 'productDetail'})
          this.orderData.total_fee = this.totalPrice;
          this.orderData.product_id = this.detail.id;
          this.orderData.goodsNum = this.valueTextNum;
          this.orderData.image = `http://tianyi.zhongkakeji.com/media/${this.detail.image}`
          this.orderData.user_id = this.userInfo.user_id;
          console.log(this.orderData);
          this.$emit('orderInfo',[this.orderData]);
        }

      },
      loginFn(val){
        this.phoneLoginShow = false;
        this.userInfo = UserInfo();
      },
      closeLogin(){
        this.phoneLoginShow = false;
      },
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .myShopOrder{}
  .productOrderList{width:100%;height: auto;overflow: hidden; border-bottom: 10px solid rgba(247,247,247,1);}
  .proLeft{display: block;float: left; width: 80px;height: 80px;border: 1px solid rgba(247,247,247,1);margin: 20px;}

  .proCenter{float: left;}
  .name{font-size: 16px;color:rgba(0,0,0,1);margin: 20px 0 4px;}

  .proRight{float: right;width: 80px;}
  .allPrice{font-size: 16px;margin: 20px 0 4px;color:rgba(0,0,0,1);}
  .price{font-size: 14px; color: #797979;text-decoration: line-through;}

  .chooseDesc{margin-bottom: 70px;}
  .chooseMain{padding-left: 10px;}
  .chooseMain .name{font-size: 16px;color:rgba(0,0,0,1);height: 40px;line-height: 40px;margin-top: 10px;}
  .chooseOne{display: inline-block; border:1px solid rgba(232,232,232,1);border-radius: 4px;padding: 6px 15px;margin-right: 10px;}
  .chooseOne i{display: block;width: 25px;height: 25px;border-radius: 50%;margin: 0 auto 6px;}
  .chooseOne em{font-style: normal}
  .chooseOne.active{border:1px solid rgba(225,70,59,1);}

  /* 加减数量*/
  /*大盒子的样式*/
  .chooseCountNum{width: 130px;height: 28px;line-height: 28px;font-size: 16px;text-align: center;border:1px solid rgba(186,186,186,1);border-radius: 5px;}
  .valueText{width: 40px;border: none;float: left;text-align: center;}
  /*加减按钮一边一个*/
  .minleft{display: inline-block; width: 44px;float: left;border-radius: 5px 0 0 5px;}
  .maxright{display: inline-block; width: 44px;float: right;border-radius: 0 5px 5px 0;}
  /*加减按钮的样式*/
  .minMax{display: inline-block;width: 40px;border: 0;cursor: pointer;}
  /*中间按钮的样式*/
  .valueText{float: left;width:48px;border: 0;text-align: center;font-size: 14px;color: #666;background-color: rgba(247,247,247,1);border-left:1px solid rgba(186,186,186,1);border-right:1px solid rgba(186,186,186,1);}
  .addActive{background-color:rgba(247,247,247,1)}

  .productDetailScreen{height:100%;overflow: scroll;}
  .footerBtn{width: 100%; position: fixed;bottom: 0;left: 0;}
  .loadPosition{position: fixed;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);}
</style>
