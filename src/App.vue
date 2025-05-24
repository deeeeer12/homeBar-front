<script>
import config from '@/config.js'
import Vue from 'vue'

Vue.prototype.$eventBus = new Vue()  // 添加全局事件总线

export default {
  onLaunch() {
    console.log('App Launch');
    console.log(config.BASE_URL)

    // 使用 uni.login 进行微信小程序登录
    uni.login({
      provider: 'weixin',
      success: (loginRes) => {
        const code = loginRes.code;
        console.log('获取到的 code:', code);

        // 调用后端登录接口
        uni.request({
          url: `${config.BASE_URL}/homebar/client/api/wxLogin`,
          method: 'POST',
          data: { code },
          success: (res) => {
            console.log('后端返回：', res.data);

            if (res.data.status === "200") {
              const token = res.data.token;
              const openid = res.data.openid;

              uni.setStorageSync('token', token);
              uni.setStorageSync('userId', openid);

              // 登录成功后广播事件
              this.$eventBus.$emit('login-success');

              console.log('登录成功，token 已保存');
            } else if (res.data.status === "404") {
              const openid = res.data.openid;
              console.log('用户未注册，跳转注册页面');
              uni.navigateTo({
                url: `/pages/auth/index?openid=${encodeURIComponent(openid)}`
              });
            } else {
              console.error('登录失败：', res.data.msg);
              uni.showToast({
                title: '登录失败',
                icon: 'none'
              });
            }
          },
          fail: (err) => {
            console.error('请求登录接口失败：', err);
            uni.showToast({
              title: '网络异常',
              icon: 'none'
            });
          }
        });
      },
      fail: (err) => {
        console.error('uni.login 失败：', err);
        uni.showToast({
          title: '微信登录失败',
          icon: 'none'
        });
      }
    });
  },

  onShow() {
    console.log('App Show');
  },

  onHide() {
    console.log('App Hide');
  }
};
</script>


<style>
/* 每个页面公共 css */
</style>
