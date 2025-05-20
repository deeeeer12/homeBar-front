<template>
  <view class="mine-container">
    <view class="mine-header">
      <text class="mine-title">👤 我的订单</text>
    </view>

    <view v-if="orders.length > 0" class="order-list">
      <view v-for="order in orders" :key="order.id" class="order-card">
        <view class="order-meta">
          <text class="order-id">订单号：{{ order.id }}</text>
          <text class="order-time">下单时间：{{ order.createdAt }}</text>
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
      <text>🕳 暂无订单信息</text>
      <button type="primary" @click="goToMenu">去点单</button>
    </view>

    <!-- ✅ 管理员入口按钮 -->
   <view v-if="isAdmin" class="admin-entry">
  <button class="admin-button" @click="goToAdmin">进入管理后台</button>
</view>
  </view>
  
</template>

<script>
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
        url: `http://localhost:8080/homebar/client/api/getOrder?openId=${openId}`,
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
        url: `http://localhost:8080/homebar/client/api/getUserRole?openId=${openId}`,
        method: 'GET',
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
  background-color: #f7f8fa;
  min-height: 100vh;
  box-sizing: border-box;
}

.mine-header {
  text-align: center;
  margin-bottom: 40rpx;
}

.mine-title {
  font-size: 44rpx;
  font-weight: bold;
  color: #1f1f1f;
}

.order-list {
  display: flex;
  flex-direction: column;
  gap: 32rpx;
}

.order-card {
  background-color: #ffffff;
  border-radius: 24rpx;
  padding: 28rpx;
  box-shadow: 0 6rpx 16rpx rgba(0, 0, 0, 0.05);
}

.order-meta {
  border-bottom: 1rpx solid #f0f0f0;
  padding-bottom: 16rpx;
  margin-bottom: 16rpx;
}

.order-id,
.order-time {
  display: block;
  font-size: 28rpx;
  color: #595959;
  margin-bottom: 8rpx;
}

.admin-entry {
  text-align: center;
  margin-top: 60rpx;
}

.admin-button {
  background-color: #ff4d4f;
  color: #fff;
  padding: 24rpx 40rpx;
  border-radius: 32rpx;
  font-size: 32rpx;
  font-weight: bold;
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
  color: #262626;
}

.dish-name {
  font-weight: 500;
}

.dish-qty {
  color: #999;
}

.empty {
  text-align: center;
  margin-top: 120rpx;
  font-size: 30rpx;
  color: #8c8c8c;
}

.dish-wrapper {
  background-color: #f5f7fa;
  border-radius: 20rpx;
  padding: 24rpx;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.04);
}

.dish-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #444;
  margin-bottom: 16rpx;
  display: block;
}

button[type="primary"] {
  margin-top: 30rpx;
  background-color: #1677ff;
  border-radius: 24rpx;
  font-size: 30rpx;
  padding: 16rpx 40rpx;
  color: #fff;
}
</style>
