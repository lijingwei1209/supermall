<template>
  <div class="goods-item" @click="itemClick">
    <!-- <img :src="showImage" alt=""  @load="imageLoad"/> -->
    <!-- 懒加载 -->
    <img v-lazy="showImage" alt="">
    <div class="goods-info">
      <p>{{ product.title }}</p>
      <span class="price">¥{{ product.price }}</span>
      <span class="collect">{{ product.cfav }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "GoodListItem",
  props: {
    product: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  computed:{
    showImage(){
      return this.product.image ||this.product.show.img
    }
  },
  methods:{
    imageLoad(){
      //运用事件总线来通信(兄弟组件之间/非父子组件的通信)
      
      this.$bus.$emit('itemImageLoad')
    },
    itemClick(){
      this.$router.push('/detail/'+this.product.iid)
    }
  }
};
</script>

<style scoped>
.goods-item {
  padding-bottom: 40px;
  position: relative;
  width: 48%;
}
.goods-item img {
  width: 100%;
  border-radius: 5px;
}

.goods-info {
  font-size: 12px;
  position: absolute;
  bottom: 5px;
  left: 0;
  right: 0;
  overflow: hidden;
  text-align: center;
}

.goods-info p {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-bottom: 3px;
}

.goods-info .price {
  color: var(--color-high-text);
  margin-right: 20px;
}

.goods-info .collect {
  position: relative;
}

.goods-info .collect::before {
  content: "";
  position: absolute;
  left: -15px;
  top: 0;
  width: 14px;
  height: 14px;
  background: url("~assets/img/common/collect.svg") 0 0/14px 14px;
}
</style>
