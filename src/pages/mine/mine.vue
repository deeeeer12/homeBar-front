<template>
  <view class="mine-container">
    <view class="mine-header">
      <image class="avatar" :src="avatarUrl" mode="aspectFill"></image>
      <text class="mine-title">我的订单</text>
    </view>

    <view v-if="orders && orders.length > 0" class="order-list">
      <view v-for="order in orders" :key="order.id" class="order-card">
        <view class="order-meta">
          <text class="order-id">订单号：{{ order.id }}</text>
          <text class="order-time">下单时间：{{ order.createdAt.replace('T',' ') }}</text>
          <text class="order-status">订单状态：{{ order.status }}</text>
        </view>
        <view class="dish-wrapper">
        <view class="dish-list">
          <text class="dish-title">单品：</text>
          <view v-for="item in order.items" :key="item.dishId" class="dish-item">
            <text class="dish-name">{{ item.dishName }}</text>
          </view>
        </view>
        </view>
      </view>
    </view>

    <view v-else class="empty">
      <text>❌暂无订单信息</text>
      <button type="primary" @click="goToMenu">去点单</button>
    </view>

    <!-- ✅ 管理员入口按钮 -->
   <view v-if="isAdmin" class="admin-entry">
  <button class="admin-button" @click="goToAdmin">进入管理后台</button>
</view>
  </view>
  
</template>

<script>
import config from '@/config.js'
export default {
  data() {
    return {
      orders: [],
      isAdmin: false
    }
  },
  methods: {
    fetchOrders() {
      const openId = uni.getStorageSync('userid')
      if (!openId) {
        uni.showToast({ title: '用户未登录', icon: 'none' })
        return
      }

      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/getOrder?openId=${openId}`,
        method: 'GET',
        header: {
          'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            this.orders = res.data.data
          } else {
            uni.showToast({ title: res.data.msg || '获取订单失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '网络错误', icon: 'none' })
        }
      })
    },    
    fetchUserInfo() {
      const openId = uni.getStorageSync('userid')
      if (!openId) return
      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/getUserRole?openId=${openId}`,
        method: 'GET',
        header: {
            'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            console.log(res.data.data)
            this.isAdmin = res.data.data === 'admin'
          }
        }
      })
    },
    goToAdmin() {
      uni.navigateTo({
        url: '/pages/admin/admin'
      })
    },
    goToMenu() {
      uni.switchTab({
        url: '/pages/index/index'
      })
    }
  },
  onShow() {
    this.fetchOrders()
    this.fetchUserInfo()
  }
}
</script>

<style scoped>
.mine-container {
  padding: 32rpx;
  background: linear-gradient(135deg, #1a1a1a 0%, #2c2c2c 100%);
  min-height: 100vh;
  box-sizing: border-box;
}

.mine-header {
  text-align: center;
  margin-bottom: 40rpx;
  padding: 20rpx 0;
  border-bottom: 2rpx solid rgba(255, 255, 255, 0.1);
}

.mine-title {
  font-size: 48rpx;
  font-weight: bold;
  color: #ffd700;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
  letter-spacing: 4rpx;
}

.order-list {
  display: flex;
  flex-direction: column;
  gap: 32rpx;
}

.order-card {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 24rpx;
  padding: 28rpx;
  box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  transition: transform 0.3s ease;
}

.order-card:active {
  transform: scale(0.98);
}

.order-meta {
  border-bottom: 1rpx solid rgba(255, 255, 255, 0.1);
  padding-bottom: 16rpx;
  margin-bottom: 16rpx;
}

.order-id,
.order-time,
.order-status {
  display: block;
  font-size: 28rpx;
  color: #b8b8b8;
  margin-bottom: 8rpx;
  line-height: 1.6;
}

.order-status {
  color: #ffd700;
  font-weight: bold;
}

.dish-wrapper {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 20rpx;
  padding: 24rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.1);
  border: 1rpx solid rgba(255, 255, 255, 0.05);
}

.dish-list {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.dish-item {
  display: flex;
  justify-content: space-between;
  font-size: 30rpx;
  color: #b8b8b8;
  padding: 8rpx 0;
  border-bottom: 1rpx solid rgba(255, 255, 255, 0.05);
}

.dish-item:last-child {
  border-bottom: none;
}

.dish-name {
  font-weight: 500;
  color: #ffd700;
}

.dish-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #ffd700;
  margin-bottom: 20rpx;
  display: block;
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

.empty {
  text-align: center;
  margin-top: 120rpx;
  font-size: 32rpx;
  color: #b8b8b8;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 40rpx;
}

.empty text {
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.3);
}

button[type="primary"] {
  margin-top: 30rpx;
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

.admin-entry {
  text-align: center;
  margin-top: 60rpx;
  padding: 20rpx;
}

.admin-button {
  background: linear-gradient(45deg, #ff4d4f, #d9363e);
  color: #fff;
  padding: 24rpx 40rpx;
  border-radius: 32rpx;
  font-size: 32rpx;
  font-weight: bold;
  box-shadow: 0 4rpx 12rpx rgba(255, 77, 79, 0.3);
  border: none;
  transition: transform 0.3s ease;
}

.admin-button:active {
  transform: scale(0.98);
}
</style>
