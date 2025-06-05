<template>
  <view class="cart-container">
    <view class="cart-header">
      <text class="cart-title">🛒 我的点单</text>
    </view>

    <view v-if="Array.isArray(cart) && cart.length > 0" class="cart-list">
      <view v-for="item in cart" :key="item.id" class="cart-item">
        <image :src="item.imageUrl" class="item-image" mode="aspectFill" />
        <view class="item-info">
          <text class="item-name">{{ item.name }}</text>
          <text class="item-desc">{{ item.description }}</text>
        </view>
        <button class="delete-btn" @click="deleteDish(item.id)" aria-label="删除">
          <text class="delete-icon">🙅</text>
        </button>
      </view>

      <view class="order-button-container">
        <button type="primary" @click="submitOrder">提交订单</button>
      </view>
    </view>

    <view v-else class="empty">
      <text>🕳 暂无菜品，请先去选择！</text>
      <button type="primary" @click="goToMenu">继续点餐</button>
      <button type="primary" @click="goToMine">查看订单</button>
    </view>
  </view>
</template>

<script>
import config from '@/config.js'
export default {
  data() {
    return {
      cart: []
    }
  },
  methods: {
    fetchCartData() {
      const openId = uni.getStorageSync('userId')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/getCartInfo?openId=${openId}`,
        method: 'GET',
        header: {
          'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            this.cart = res.data.data
          } else {
            uni.showToast({ title: res.data.msg || '获取失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '网络错误', icon: 'none' })
        }
      })
    },
    submitOrder() {
      const openId = uni.getStorageSync('userId')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/takeOrder?userid=${openId}`,
        method: 'POST',
        header: {
          'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            uni.showToast({ title: res.data.msg || '下单成功', icon: 'success' })
            this.fetchCartData()
          } else {
            uni.showToast({ title: res.data.msg || '下单失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '网络错误', icon: 'none' })
        }
      })
    },
    deleteDish(dishId) {
      const openId = uni.getStorageSync('userId')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/delCartDish`,
        method: 'POST',
        data: {
          openId: openId,
          dishId: dishId
        },
        header: {
          'Authorization': uni.getStorageSync('token'),
          'Content-Type': 'application/json'
        },
        success: (res) => {
          if (res.data.status === 200) {
            uni.showToast({ title: res.data.msg, icon: 'success' })
            this.fetchCartData()
          } else {
            uni.showToast({ title: res.data.msg || '删除失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '网络错误', icon: 'none' })
        }
      })
    },
    goToMenu() {
      uni.switchTab({
        url: '/pages/index/index'
      })
    },
    goToMine() {
      uni.switchTab({
        url: '/pages/mine/mine'
      })
    }
  },
  onShow() {
    this.fetchCartData()
  }
}
</script>

<style scoped>
.cart-container {
  padding: 32rpx;
  background: linear-gradient(135deg, #1a1a1a 0%, #2c2c2c 100%);
  min-height: 100vh;
  box-sizing: border-box;
}

.cart-header {
  text-align: center;
  margin-bottom: 40rpx;
  padding: 20rpx 0;
  border-bottom: 2rpx solid rgba(255, 255, 255, 0.1);
}

.cart-title {
  font-size: 48rpx;
  font-weight: bold;
  color: #ffd700;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
  letter-spacing: 4rpx;
}

.cart-list {
  display: flex;
  flex-direction: column;
  gap: 24rpx;
}

.cart-item {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 24rpx;
  padding: 24rpx;
  box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, 0.2);
  gap: 24rpx;
  backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  transition: transform 0.3s ease;
}

.cart-item:active {
  transform: scale(0.98);
}

.item-image {
  width: 160rpx;
  height: 160rpx;
  border-radius: 16rpx;
  object-fit: cover;
  background-color: rgba(255, 255, 255, 0.1);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
}

.item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 12rpx;
}

.item-name {
  font-size: 34rpx;
  font-weight: 600;
  color: #ffd700;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.item-desc {
  font-size: 28rpx;
  color: #b8b8b8;
  line-height: 1.4;
}

.delete-btn {
  background: rgba(255, 77, 79, 0.1);
  border: 1.5rpx solid #ff4d4f;
  border-radius: 12rpx;
  padding: 12rpx 20rpx;
  color: #ff4d4f;
  font-size: 28rpx;
  line-height: 28rpx;
  cursor: pointer;
  transition: all 0.3s ease;
  user-select: none;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(5px);
}

.delete-btn:hover {
  background: rgba(255, 77, 79, 0.2);
  transform: scale(1.05);
}

.delete-btn:active {
  background: rgba(255, 77, 79, 0.3);
  transform: scale(0.95);
}

.order-button-container {
  margin-top: 48rpx;
  text-align: center;
}

button[type="primary"] {
  background: linear-gradient(45deg, #ffd700, #ffa500);
  border-radius: 24rpx;
  font-size: 32rpx;
  padding: 20rpx 48rpx;
  color: #000;
  font-weight: bold;
  box-shadow: 0 4rpx 12rpx rgba(255, 215, 0, 0.3);
  border: none;
  transition: transform 0.3s ease;
}

button[type="primary"]:active {
  transform: scale(0.98);
}

.empty {
  text-align: center;
  margin-top: 120rpx;
  color: #b8b8b8;
  font-size: 32rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 40rpx;
}

.empty text {
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.empty button {
  margin-top: 20rpx;
  width: 80%;
  max-width: 400rpx;
}

.empty button + button {
  margin-top: 20rpx;
  background: linear-gradient(45deg, #4a4a4a, #2c2c2c);
  color: #ffd700;
  border: 1rpx solid rgba(255, 215, 0, 0.3);
}
</style>
