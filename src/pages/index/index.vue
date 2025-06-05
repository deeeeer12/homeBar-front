<template>
  <view class="container">
    <view class="header">
      <text class="title">Glu's homeBar</text>
      <text class="subtitle"></text>
    </view>

    <!-- 分类 Tab -->
    <view class="tab-bar">
      <view
        v-for="(item, index) in categories"
        :key="index"
        class="tab-item"
        :class="{ active: currentTab === index }"
        @tap="changeTab(index)"
      >
        {{ item }}
      </view>
    </view>

    <!-- 菜品列表 -->
    <view class="dish-list">
      <view
        v-for="dish in filteredDishes"
        :key="dish.id"
        class="dish-card"
        @tap="showDishDetail(dish.id)"
      >
        <image :src="dish.image" class="dish-img" mode="aspectFill" />
        <view class="dish-info">
          <text class="dish-name">{{ dish.name }}</text>
          <button class="add-btn" type="primary" size="mini" @tap.stop="addToCart(dish)">
            加入点单
          </button>
        </view>
      </view>
    </view>

    <!-- 购物车悬浮按钮 -->
    <view class="floating-cart-btn" @tap="goToCart">
      🛒
    </view>

    <!-- 弹窗：酒品详情 -->
    <view v-if="showModal" class="modal-mask" @tap.self="closeModal">
      <view class="modal-content">
        <image :src="selectedDishDetail.dish.imageUrl" mode="aspectFill" class="modal-img" />
        <view class="modal-body">
          <text class="modal-title">{{ selectedDishDetail.name }}</text>
          <view class="modal-section">
            <text class="modal-label">酒精度：</text>
            <text class="modal-text">{{ selectedDishDetail.alcoholContent }}%</text>
          </view>
          <view class="modal-section">
            <text class="modal-label">配方：</text>
            <text class="modal-text">{{ selectedDishDetail.recipe }}</text>
          </view>
          <view class="modal-section">
            <text class="modal-label">简介：</text>
            <text class="modal-text">{{ selectedDishDetail.dish.description }}</text>
          </view>

          <button type="primary" class="modal-order-btn" @tap="placeOrder">立即下单</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import config from '@/config.js'
export default {
  data() {
    return {
      categories: ['全部', '经典', 'Jay', 'JJ', '陶喆', '网红款'],
      currentTab: 0,
      dishes: [],
      showModal: false,
      selectedDishDetail: {}
    }
  },
  computed: {
    filteredDishes() {
      const category = this.categories[this.currentTab]
      if (category === '全部') {
        return this.dishes
      }
      const typeMap = {
        '经典': 1,
        'Jay': 2,
        'JJ': 3,
        '陶喆': 4,
        '网红款': 5
      }
      return this.dishes.filter(d => d.type === typeMap[category])
    }
  },
  methods: {
    changeTab(index) {
      this.currentTab = index
    },
    addToCart(dish) {
      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/addToCart`,
        method: 'POST',
        header: {
          'Authorization': uni.getStorageSync('token'),
          'Content-Type': 'application/json'
        },
        data: {
          dishId: dish.id,
          openId: uni.getStorageSync('userId')
        },
        success: (res) => {
          if (res.data.status === 200) {
            uni.showToast({
              title: dish.name + ' +1',
              icon: 'success'
            })
          } else {
            uni.showToast({
              title: res.data.msg || '添加失败',
              icon: 'none'
            })
          }
        },
        fail: () => {
          uni.showToast({
            title: '网络错误',
            icon: 'none'
          })
        }
      })
    },
    placeOrder() {
      const dish = this.selectedDishDetail.dish;
      if (!dish || !dish.id) return;
    
      this.addToCart(dish); // 调用原来的添加接口
      this.closeModal(); // 关闭弹窗
    
      // 可选：防止没有响应 toast
      uni.showToast({
        title: dish.name + ' 已下单',
        icon: 'success'
      });
    },
    goToCart() {
      uni.navigateTo({
        url: '/pages/cart/cart'
      })
    },
    fetchDishes() {
      const token = uni.getStorageSync('token');
      if (!token) {
        console.log('等待登录完成...');
        return;
      }

      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/getAllDishes`,
        method: 'GET',
        header: {
          'Authorization': token
        },
        success: (res) => {
          if (res.data.status === 200) {
            this.dishes = res.data.data.map(d => ({
              id: d.id,
              name: d.name,
              category: d.category,
              type: d.type,
              image: d.imageUrl
            }))
          }
        },
        fail: () => {
          uni.showToast({ title: '获取菜单失败', icon: 'none' })
        }
      })
    },
    showDishDetail(dishId) {
      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/getDishInfoById?dishId=${dishId}`,
        method: 'GET',
        header: {
          'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            this.selectedDishDetail = res.data.data
            this.showModal = true
          } else {
            uni.showToast({ title: '加载详情失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '请求失败', icon: 'none' })
        }
      })
    },
    closeModal() {
      this.showModal = false
    }
  },
  onLoad() {
    // 监听登录成功事件
    this.$eventBus.$on('login-success', () => {
      console.log('登录成功，开始获取餐品信息');
      this.fetchDishes();
    });

    // 如果已经登录，直接获取餐品信息
    const token = uni.getStorageSync('token');
    if (token) {
      this.fetchDishes();
    }
  },
  onUnload() {
    // 页面卸载时移除事件监听
    this.$eventBus.$off('login-success');
  }
}
</script>

<style scoped>
.container {
  padding: 24rpx;
  background: linear-gradient(135deg, #1a1a1a 0%, #2c2c2c 100%);
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 40rpx;
  padding: 20rpx 0;
  border-bottom: 2rpx solid rgba(255, 255, 255, 0.1);
  position: relative;
}

.header::before {
  content: '';
  position: absolute;
  top: -10rpx;
  left: 50%;
  transform: translateX(-50%);
  width: 60rpx;
  height: 4rpx;
  background: linear-gradient(90deg, transparent, #ffd700, transparent);
}

.title {
  font-family: "Times New Roman", "Georgia", serif;
  font-size: 56rpx;
  font-weight: 700;
  font-style: italic;
  color: #ffd700;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3),
               0 4rpx 8rpx rgba(0, 0, 0, 0.2);
  letter-spacing: 6rpx;
  position: relative;
  display: inline-block;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -10rpx;
  left: 50%;
  transform: translateX(-50%);
  width: 80%;
  height: 2rpx;
  background: linear-gradient(90deg, transparent, #ffd700, transparent);
}

.subtitle {
  font-family: "Times New Roman", "Georgia", serif;
  font-size: 28rpx;
  color: #b8b8b8;
  margin-top: 16rpx;
  letter-spacing: 4rpx;
  font-style: italic;
}

.tab-bar {
  display: flex;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.05);
  padding: 20rpx;
  margin: 20rpx 0 40rpx;
  border-radius: 20rpx;
  backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  flex-wrap: wrap;
  gap: 16rpx;
  position: relative;
  overflow: hidden;
}

.tab-bar::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, 
    transparent,
    rgba(255, 215, 0, 0.3),
    transparent
  );
  animation: shimmer 2s infinite;
}

.tab-bar::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, 
    transparent,
    rgba(255, 215, 0, 0.3),
    transparent
  );
  animation: shimmer 2s infinite reverse;
}

@keyframes shimmer {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

.tab-item {
  padding: 16rpx 24rpx;
  font-size: 26rpx;
  border-radius: 40rpx;
  color: #b8b8b8;
  background: rgba(255, 255, 255, 0.05);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  white-space: nowrap;
  flex: 1;
  text-align: center;
  min-width: 120rpx;
  position: relative;
  overflow: hidden;
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(5px);
}

.tab-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, 
    rgba(255, 215, 0, 0.1),
    rgba(255, 165, 0, 0.1)
  );
  opacity: 0;
  transition: opacity 0.3s ease;
}

.tab-item:active {
  transform: scale(0.95);
}

.tab-item.active {
  background: linear-gradient(45deg, #ffd700, #ffa500);
  color: #000;
  font-weight: bold;
  box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
  border: none;
  animation: pulse 2s infinite;
}

.tab-item.active::before {
  opacity: 1;
}

@keyframes pulse {
  0% {
    box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
  }
  50% {
    box-shadow: 0 4rpx 20rpx rgba(255, 215, 0, 0.5);
  }
  100% {
    box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
  }
}

.tab-item:hover::before {
  opacity: 0.5;
}

/* 添加滚动条样式 */
::-webkit-scrollbar {
  width: 6rpx;
  height: 6rpx;
}

::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3rpx;
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(45deg, #ffd700, #ffa500);
  border-radius: 3rpx;
}

::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(45deg, #ffa500, #ffd700);
}

.dish-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  padding: 10rpx;
}

.dish-card {
  width: 48%;
  background: rgba(255, 255, 255, 0.05);
  margin-bottom: 30rpx;
  border-radius: 24rpx;
  overflow: hidden;
  box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  transition: transform 0.3s ease;
}

.dish-card:active {
  transform: scale(0.98);
}

.dish-img {
  width: 100%;
  height: 360rpx;
  object-fit: cover;
  border-bottom: 1rpx solid rgba(255, 255, 255, 0.1);
}

.dish-info {
  padding: 24rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.dish-name {
  font-size: 32rpx;
  font-weight: bold;
  color: #ffd700;
  text-align: center;
  margin-bottom: 16rpx;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.add-btn {
  margin-top: 20rpx;
  width: 100%;
  border-radius: 16rpx;
  background: linear-gradient(45deg, #ffd700, #ffa500);
  color: #000;
  font-size: 26rpx;
  font-weight: bold;
  border: none;
  box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
}

.floating-cart-btn {
  position: fixed;
  bottom: 80rpx;
  right: 40rpx;
  width: 110rpx;
  height: 110rpx;
  background: linear-gradient(45deg, #ffd700, #ffa500);
  color: #000;
  font-size: 48rpx;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 8rpx 24rpx rgba(255, 215, 0, 0.4);
  z-index: 999;
  border: 2rpx solid rgba(255, 255, 255, 0.2);
}

/* 弹窗样式 */
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(8px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  width: 85%;
  background: rgba(28, 28, 28, 0.95);
  border-radius: 32rpx;
  overflow: hidden;
  box-shadow: 0 16rpx 40rpx rgba(0, 0, 0, 0.4);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
}

.modal-img {
  width: 100%;
  height: 360rpx;
  object-fit: cover;
  border-bottom: 1rpx solid rgba(255, 255, 255, 0.1);
}

.modal-body {
  padding: 36rpx 28rpx;
}

.modal-title {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffd700;
  text-align: center;
  margin-bottom: 30rpx;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.modal-section {
  margin-bottom: 24rpx;
  padding: 16rpx;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 16rpx;
}

.modal-label {
  font-weight: bold;
  color: #ffd700;
  font-size: 28rpx;
  margin-bottom: 8rpx;
}

.modal-text {
  font-size: 26rpx;
  color: #b8b8b8;
  line-height: 40rpx;
}

.modal-order-btn {
  width: 100%;
  padding: 24rpx 0;
  font-size: 32rpx;
  border-radius: 32rpx;
  text-align: center;
  background: linear-gradient(45deg, #ffd700, #ffa500);
  color: #000;
  font-weight: bold;
  margin-top: 40rpx;
  box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
  border: none;
}
</style>
