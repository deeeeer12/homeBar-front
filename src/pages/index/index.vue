<template>
  <view class="container">
    <view class="menu-item" v-for="item in menuList" :key="item.id">
      <text>{{ item.name }} - ¥{{ item.price }}</text>
      <button @click="addToCart(item)">加入购物车</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      menuList: []
    };
  },
  methods: {
    addToCart(item) {
      uni.$emit('addToCart', item);
      uni.showToast({ title: '已加入购物车' });
    }
  },
  onLoad() {
    // 模拟请求菜单
    this.menuList = [
      { id: 1, name: '红烧牛肉面', price: 18 },
      { id: 2, name: '鸡腿饭', price: 22 },
      { id: 3, name: '酸辣土豆丝', price: 10 }
    ];
  }
};
</script>

<style>
.container {
  padding: 20rpx;
}
.menu-item {
  margin-bottom: 20rpx;
}
</style>