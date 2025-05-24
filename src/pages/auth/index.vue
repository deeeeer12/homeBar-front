<template>
  <view class="auth-container">
    <view class="logo">🍸 欢迎来到 HOME BAR</view>

    <button class="auth-btn" open-type="getUserInfo" @getuserinfo="onGetUserInfo">
      授权登录获取头像昵称
    </button>
  </view>
</template>

<script>
import config from '@/config.js'
export default {
  onLoad(options) {
    this.openid = options.openid || '';
  },
  data() {
    return {
      openid: ''
    };
  },
  methods: {
    async onGetUserInfo(e) {
      const userInfo = e.detail.userInfo;
      console.log(userInfo)
      if (!userInfo) {
        uni.showToast({ title: '授权失败', icon: 'none' });
        return;
      }

      // 向后端提交用户信息+openid
      const res = await uni.request({
        url: `${config.BASE_URL}/homebar/client/api/register`,
        method: 'POST',
        data: {
          openId: this.openid,
          nickName: userInfo.nickName,
          avatarUrl: userInfo.avatarUrl,
          // gender: userInfo.gender
        }
      });

      if (res.data.status === '200') {
        uni.setStorageSync('token', res.data.token);
        uni.setStorageSync('avatarUrl',userInfo.avatarUrl)
        uni.setStorageSync('userId', this.openid);
        uni.showToast({ title: '登录成功' });

        // 跳转首页
        uni.switchTab({ url: '/pages/index/index' });
      } else {
        uni.showToast({ title: '注册失败', icon: 'none' });
      }
    }
  }
};
</script>

<style>
.auth-container {
  padding: 100rpx 40rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.logo {
  font-size: 40rpx;
  margin-bottom: 80rpx;
  font-weight: bold;
}
.auth-btn {
  background-color: #07c160;
  color: white;
  font-size: 32rpx;
  padding: 20rpx 40rpx;
  border-radius: 10rpx;
}
</style>
