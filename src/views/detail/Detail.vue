<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="detailnav">
    </detail-nav-bar>
    <com-scroll class="content"  ref="scroll"
      :probe-type="3" @scroll="contentScroll"
      >
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <detail-goods-info :detailInfo="detailInfo" @imageLoad="imageLoad"></detail-goods-info>
      <detail-param-info :paramInfo="paramInfo"  ref="param"></detail-param-info>
      <detail-comment-info :commentInfo="commentInfo" ref="comment"></detail-comment-info>
      <good-list :goods="recommends" ref="recommend"></good-list>
    </com-scroll>
      <!-- 添加native修饰符，监听组件根源上的原生事件 -->
    <back-top @click.native="backTop" v-show="isShowBsckTop"></back-top>
     <detail-bottom-bar @addCart="addCartlist"></detail-bottom-bar>
     <toast :message="message" :show="show"></toast>
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar'
import DetailSwiper from './childComps/DetailSwiper'
import DetailBaseInfo from './childComps/DetailBaseInfo'
import DetailShopInfo from './childComps/DetailShopInfo'
import DetailGoodsInfo from './childComps/DetailGoodsInfo'
import DetailParamInfo from './childComps/DetailParamInfo'
import DetailCommentInfo from './childComps/DetailCommentImfo'
import DetailBottomBar from './childComps/DetailBottomBar'

import ComScroll from "components/common/scroll/ComScroll";
import GoodList from "components/content/goods/GoodList";
import Toast from 'components/common/toast/Toast';


import  {getDetail,Goods,Shop,GoodsParam,getRecommend} from 'network/detail'

import {debounce} from 'common/utils'
import {itemListenerMixin,BackTopMixin} from 'common/mixin'


import {mapActions} from 'vuex'
	export default {
    name: "Detail",
    data(){
      return{
        id:null,
        topImages:[],
        goods:{},
        shop:{},
        detailInfo:{},
        paramInfo:{},
        commentInfo:{},
        recommends:[],
        themeTopYs:[],
        currentIndex:0,
        show:false,
        message:''
      }
    },
    mixins:[itemListenerMixin,BackTopMixin],
    components:{
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      ComScroll,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      GoodList,
      DetailBottomBar,
      Toast
    },
    created(){
      this.id = this.$route.params.id;

      //根绝iid获取详情页数据
      getDetail(this.id).then(res=>{
        //获取顶部图片轮播数据
        const data = res.result

        this.topImages = data.itemInfo.topImages;
        //2.获取商品信息
        this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
        //3.获取店铺的信心
        this.shop = new Shop(data.shopInfo)
        //4. 获取商品详情数据
        this.detailInfo = data.detailInfo
        //5.获取参数
        this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)
        //6.取出评论信息
        if(data.rate.cRate!==0){
          this.commentInfo = data.rate.list[0]
        }
      })

      //获取推荐数据
      getRecommend().then(res=>{
        this.recommends= res.data.list
      })

      
    },
    mounted(){
         console.log("mounted")
      //监听推荐组件中图片的加载完成
      // const newRefresh = debounce(this.$refs.scroll.refresh,500)

      // this.itemImgListener = ()=>{
      //   newRefresh()
      // }
      // this.$bus.$on('itemImageLoad',this.itemImgListener)
   
    },
    methods:{
      ...mapActions(['addCart']),
      imageLoad(){
        this.$refs.scroll.refresh();
        this.themeTopYs=[]
        this.themeTopYs.push(0);
       // console.log(this.$refs.param.$el.offsetTop);
        this.themeTopYs.push(this.$refs.param.$el.offsetTop);
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
        //console.log("this.themeTopYs",this.themeTopYs,Number.MAX_VALUE);
        this.themeTopYs.push(Number.MAX_VALUE)
      },
      titleClick(index){
         this.$refs.scroll.cScrollTo(0, -this.themeTopYs[index], 100)
      },
      contentScroll(position){
        // 获取y值
        const positionY = -position.y
        //positiony和主题中的值进行对比
        let length = this.themeTopYs.length;
         //判断backtop是否显示
       
        for(let i = 0;i<length;i++){
            
          // if((this.currentIndex!==i)&&(i<length-1&& positionY>this.themeTopYs[i]&&positionY<this.themeTopYs[i+1])||(i==length-1&& positionY>this.themeTopYs[length-1])){
            if(this.currentIndex!==i && (positionY>this.themeTopYs[i]&&positionY<this.themeTopYs[i+1])){
              this.currentIndex = i;
               console.log(this.currentIndex,i);
              this.$refs.detailnav.currentIndex = this.currentIndex
          }
        }
        this.listenershowback(position);
       
      },
      addCartlist(){
        //获取购物车需要展示的信息
        const product ={}
        product.image = this.topImages[0];
        product.title = this.goods.title;
        product.desc = this.goods.desc;
        product.price = this.goods.newPrice;
        product.id = this.id
        //2.将商品信息天骄到购物车
        //this.$store.commit('addCart',product)
        // this.$store.dispatch('addCart',product).then(res=>{
        //   //操作完成具体的操作显示不同的提示，所以需要结合promise来根据不同的操作返回不同的提示
        //   console.log(res)
        // })
        //3.将store中的addCart映射到methdos中，用mapActions
        this.addCart(product).then(res=>{
          /* this.message= res;
          this.show = true;

          setTimeout(()=>{
            this.show= false,
            this.message = ''
          },1000) */
          //console.log(this.$toast)
          this.$toast.show(res,5000)
        })
      }

    },
    destroyed(){
      this.$bus.$off(this.itemImgListener)
    }
 

	}
</script>

<style scoped>
 #detail{
   position:relative;
   z-index:9;
   background-color:#fff;
   height:100vh;
 }
 .detail-nav{
   position:relative;
   z-index:9;
   background-color:#fff;
 }
 .content{
   background:#fff;
   height: calc(100% - 44px - 49px) ;
 }
 
</style>
