<template>
  <view class="admin-container">
    <view class="admin-header">
      <text class="admin-title">📦 订单管理后台</text>
    </view>

    <view v-if="orders && orders.length > 0" class="order-list">
      <view v-for="order in orders" :key="order.id" class="order-card">
        <view class="order-info">
          <text class="order-id">订单号：{{ order.id }}</text>
          <text class="order-user">用户名：{{ order.userName }}</text>
          <text class="order-time">时间：{{ order.createdAt.replace('T', ' ') }}</text>
          <text class="order-status">状态：{{ order.status }}</text>
        </view>

        <view class="dish-list">
          <text class="dish-title">餐品：</text>
          <view class="dish-tag" v-for="item in order.items" :key="item.dishId">
            {{ item.dishName }}
          </view>
        </view>

        <!-- 修改按钮 -->
        <view class="btn-group">
          <view class="update-btn" @click="openStatusModal(order)">修改订单状态</view>
          <view class="delete-btn" @click="deleteOrder(order.id)">删除订单</view>
        </view>
      </view>
    </view>

    <view v-else class="empty">
      <text>暂无订单数据</text>
    </view>

    <!-- 状态选择弹窗 -->
    <view v-if="showStatusModal" class="modal-mask">
      <view class="modal-box">
        <text class="modal-title">修改订单状态</text>
        <picker :value="statusOptions.indexOf(selectedStatus)" :range="statusOptions" @change="e => selectedStatus = statusOptions[e.detail.value]">
          <view class="picker-box">
            {{ selectedStatus || '请选择状态' }}
          </view>
        </picker>
        <view class="modal-actions">
          <button type="default" @click="cancelStatusUpdate">取消</button>
          <button type="primary" @click="submitStatusUpdate">提交</button>
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
      orders: [],
      showStatusModal: false,
      statusOptions: ['制作中', '已完成', '已挂起'],
      selectedStatus: '',
      currentOrderId: null
    }
  },
  methods: {
    fetchAllOrders() {
      uni.request({
        url: `${config.BASE_URL}/homebar/admin/api/getAllOrders`,
        method: 'GET',
        header: {
          'Authorization': uni.getStorageSync('token')
        },
        success: (res) => {
          if (res.data.status === 200) {
            this.orders = res.data.data
          } else {
            uni.showToast({ title: res.data.msg || '获取失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '网络错误', icon: 'none' })
        }
      })
    },  
    openStatusModal(order) {
      this.selectedOrder = order
      this.selectedStatus = order.status
      this.showStatusModal = true
    },
    submitStatusUpdate() {
      if (!this.selectedOrder) return
  
      uni.request({
        url: `${config.BASE_URL}/homebar/admin/api/updateOrderStatus`,
        method: 'POST',
        data: {
          orderId: this.selectedOrder.id,
          status: this.selectedStatus
        },
        header: {
          'Authorization': uni.getStorageSync('token'),
          'Content-Type': 'application/json'
        },
        success: (res) => {
          if (res.data.status === 200) {
            uni.showToast({ title: '更新成功', icon: 'success' })
            this.showStatusModal = false
            this.fetchAllOrders() // 刷新订单列表
          } else {
            uni.showToast({ title: res.data.msg || '更新失败', icon: 'none' })
          }
        },
        fail: () => {
          uni.showToast({ title: '请求失败', icon: 'none' })
        }
      })
    },
    cancelStatusUpdate() {
      this.showStatusModal = false
      this.selectedOrder = null
      this.selectedStatus = ''
    },
    deleteOrder(orderId) {
      uni.showModal({
        title: '确认删除',
        content: '确定要删除这个订单吗？删除后将无法恢复！',
        confirmText: '删除',
        confirmColor: '#ff4d4f',
        success: (res) => {
          if (res.confirm) {
            uni.request({
              url: `${config.BASE_URL}/homebar/admin/api/deleteOrder?orderId=${orderId}`,
              method: 'DELETE',
              header: {
                'Authorization': uni.getStorageSync('token')
              },
              success: (res) => {
                if (res.data.status === 200) {
                  uni.showToast({ title: '删除成功', icon: 'success' });
                  this.fetchAllOrders(); // 重新拉取订单列表
                } else {
                  uni.showToast({ title: res.data.msg || '删除失败', icon: 'none' });
                }
              },
              fail: () => {
                uni.showToast({ title: '网络错误', icon: 'none' });
              }
            });
          }
        }
      });
    }
  },
  onShow() {
    this.fetchAllOrders()
  }
}
</script>

<style scoped>
.admin-container {
  padding: 32rpx;
  background-color: #f0f2f5;
  min-height: 100vh;
  box-sizing: border-box;
}

.admin-header {
  text-align: center;
  margin-bottom: 36rpx;
}

.admin-title {
  font-size: 48rpx;
  font-weight: 700;
  color: #1677ff;
}

.order-list {
  display: flex;
  flex-direction: column;
  gap: 36rpx;
}

.order-card {
  background-color: #ffffff;
  border-radius: 24rpx;
  padding: 32rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.06);
  transition: transform 0.2s;
}
.order-card:hover {
  transform: scale(1.01);
}

.order-info text {
  display: block;
  font-size: 30rpx;
  color: #333333;
  margin-bottom: 10rpx;
  line-height: 40rpx;
}

.order-status {
  font-weight: 600;
  color: #fa541c;
}

.dish-list {
  margin-top: 16rpx;
}

.dish-title {
  font-size: 30rpx;
  font-weight: 600;
  color: #1f1f1f;
  margin-bottom: 12rpx;
  display: block;
}

.dish-tag {
  display: inline-block;
  background-color: #e6f4ff;
  border: 1rpx solid #91d5ff;
  color: #1677ff;
  font-size: 26rpx;
  padding: 10rpx 24rpx;
  border-radius: 32rpx;
  margin: 8rpx 12rpx 0 0;
}

.empty {
  text-align: center;
  color: #999999;
  font-size: 32rpx;
  margin-top: 200rpx;
}

/* 按钮美化 */
.update-btn {
  display: inline-block;
  padding: 12rpx 28rpx;
  background-color: #1677ff;
  color: #fff;
  border-radius: 12rpx;
  font-size: 28rpx;
  text-align: center;
  box-shadow: 0 4rpx 12rpx rgba(22, 119, 255, 0.3);
  transition: all 0.3s;
}
.update-btn:active {
  background-color: #0958d9;
}

/* 弹窗遮罩层 */
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.35);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

/* 弹窗容器 */
.modal-box {
  width: 80%;
  background-color: #ffffff;
  border-radius: 24rpx;
  padding: 40rpx 32rpx;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.15);
}

/* 弹窗标题 */
.modal-title {
  font-size: 36rpx;
  font-weight: 600;
  margin-bottom: 30rpx;
  text-align: center;
  color: #333;
}

/* picker 选择器显示框 */
.picker-box {
  padding: 20rpx;
  border: 2rpx dashed #1677ff;
  border-radius: 16rpx;
  margin-bottom: 30rpx;
  text-align: center;
  color: #1677ff;
  font-size: 30rpx;
  background-color: #f0faff;
}

/* 弹窗按钮区域 */
.modal-actions {
  display: flex;
  justify-content: space-around;
  gap: 24rpx;
}

.modal-actions button {
  flex: 1;
  padding: 20rpx 0;
  border-radius: 12rpx;
  font-size: 30rpx;
}

.modal-actions button[type="primary"] {
  background-color: #52c41a;
  color: #fff;
}

.modal-actions button:not([type="primary"]) {
  background-color: #f5f5f5;
  color: #333;
}

.btn-group {
  display: flex;
  gap: 20rpx;
  margin-top: 20rpx;
}

.update-btn,
.delete-btn {
  padding: 16rpx 32rpx;
  font-size: 28rpx;
  border-radius: 12rpx;
  text-align: center;
  flex-shrink: 0;
}

.update-btn {
  background-color: #1677ff;
  color: #ffffff;
}

.delete-btn {
  background-color: #ff4d4f;
  color: #ffffff;
}


</style>