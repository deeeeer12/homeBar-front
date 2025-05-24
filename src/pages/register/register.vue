<template>
  <view class="container">
    <view class="header">
      <text class="title">用户注册</text>
    </view>

    <view class="form">
      <input class="input" placeholder="请输入用户名" v-model="username" />
      <input class="input" placeholder="请输入手机号" v-model="phone" type="number" />

      <button class="register-btn" type="primary" @tap="register">
        注册
      </button>
    </view>
  </view>
</template>


<script>
import config from '@/config.js'
export default {
  data() {
    return {
      openid: '',
      username: '',
      phone: ''
    }
  },
  onLoad(options) {
    this.openid = options.openid || ''
    console.log('接收到的 openid：', this.openid)
  },
  methods: {
    register() {
      if (!this.username || !this.phone) {
        return uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        })
      }
    
      uni.request({
        url: `${config.BASE_URL}/homebar/client/api/register`,
        method: 'POST',
        data: {
          id: this.openid,
          username: this.username,
          phone: this.phone
        },
        success: (res) => {
          console.log('注册接口返回：', res)
          if (res.data.status === "200") {
            uni.setStorageSync('token', res.data.token)
            uni.setStorageSync('userid', this.openid)
    
            uni.showToast({
              title: '注册成功',
              icon: 'success',
              duration: 1000
            })
    
            // 延迟跳转到 tabBar 首页
            setTimeout(() => {
              uni.switchTab({
                url: '/pages/index/index'
              })
            }, 1000)
    
          } else {
            uni.showToast({
              title: res.data.msg || '注册失败',
              icon: 'none'
            })
          }
        },
        fail: () => {
          uni.showToast({
            title: '请求失败，请检查网络',
            icon: 'none'
          })
        }
      })
    }
  }
}

</script>

<style scoped>
.container {
  padding: 40rpx;
}

.header {
  text-align: center;
  margin-bottom: 40rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 30rpx;
}

.input {
  height: 80rpx;
  border: 1rpx solid #ccc;
  border-radius: 12rpx;
  padding: 0 20rpx;
  font-size: 28rpx;
}

.register-btn {
  margin-top: 20rpx;
}
</style>

