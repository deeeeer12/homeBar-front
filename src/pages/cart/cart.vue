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
        <button class="delete-btn" @click="deleteDish(item.id)">🗑️</button>
      </view>

      <view class="order-button-container">
        <button type="primary" @click="submitOrder">提交订单</button>
      </view>
    </view>

    <view v-else class="empty">
      <text>🕳 暂无菜品，请先去选择！</text>
      <button type="primary" @click="goToMenu">去菜单</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      cart: []
    }
  },
  methods: {
    fetchCartData() {
      const openId = uni.getStorageSync('userid')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `http://localhost:8080/homebar/client/api/getCartInfo?openId=${openId}`,
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
      const openId = uni.getStorageSync('userid')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `http://localhost:8080/homebar/client/api/takeOrder?userid=${openId}`,
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
      const openId = uni.getStorageSync('userid')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `http://localhost:8080/homebar/client/api/delCartDish`,
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
  background-color: #f2f4f5;
  min-height: 100vh;
  box-sizing: border-box;
}

.cart-header {
  text-align: center;
  margin-bottom: 40rpx;
}

.cart-title {
  font-size: 42rpx;
  font-weight: bold;
  color: #1f1f1f;
}

.cart-list {
  display: flex;
  flex-direction: column;
  gap: 24rpx;
}

.cart-item {
  display: flex;
  background-color: #ffffff;
  border-radius: 24rpx;
  padding: 24rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.06);
  align-items: center;
}

.item-image {
  width: 160rpx;
  height: 160rpx;
  border-radius: 16rpx;
  object-fit: cover;
  margin-right: 24rpx;
  background-color: #eee;
}

.item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 8rpx;
}

.item-name {
  font-size: 34rpx;
  font-weight: 600;
  color: #262626;
}

.item-desc {
  font-size: 28rpx;
  color: #8c8c8c;
}

.delete-btn {
  background-color: transparent;
  color: #ff4d4f;
  font-size: 40rpx;
  border: none;
  padding: 0 12rpx;
}

.order-button-container {
  margin-top: 48rpx;
  text-align: center;
}

button[type="primary"] {
  background-color: #1677ff;
  border-radius: 24rpx;
  font-size: 30rpx;
  padding: 16rpx 40rpx;
  color: #fff;
}

.empty {
  text-align: center;
  margin-top: 100rpx;
  color: #8c8c8c;
  font-size: 30rpx;
}

.empty button {
  margin-top: 30rpx;
}
</style>
