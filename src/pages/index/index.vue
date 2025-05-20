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
        >
          <image :src="dish.image" class="dish-img" mode="aspectFill" />
          <view class="dish-info">
            <text class="dish-name">{{ dish.name }}</text>
            <button class="add-btn" type="primary" size="mini" @tap="addToCart(dish)">
              加入点单
            </button>
          </view>
        </view>
      </view>
        <!-- 购物车悬浮按钮 -->
      <view class="floating-cart-btn" @tap="goToCart">
        🛒
      </view>
    </view>
  </template>

  <script>
  export default {
    data() {
      return {
        categories: ['全部', '主食', '饮品'],
        currentTab: 0,
        dishes: [] // 不再硬编码，接口拉取
      }
    },
    computed: {
      filteredDishes() {
        const category = this.categories[this.currentTab]
        return category === '全部'
          ? this.dishes
          : this.dishes.filter(d => d.category === category)
      }
    },
    methods: {
      changeTab(index) {
        this.currentTab = index
      },
      addToCart(dish) {
        uni.request({
          url: 'http://localhost:8080/homebar/client/api/addToCart', // 换成你的后端添加购物车接口
          method: 'POST',
          header: {
            'Authorization': uni.getStorageSync('token'),
            'Content-Type': 'application/json'
          },
          data: {
            dishId: dish.id,
            openId: uni.getStorageSync('userid') // 可以根据登录信息动态传
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
      goToCart() {
        uni.switchTab({
          url: '/pages/cart/cart'
        })
      },
      fetchDishes() {
        uni.request({
          url: 'http://localhost:8080/homebar/client/api/getAllDishes',
          method: 'GET',
          header: {
            'Authorization': uni.getStorageSync('token')
          },
          success: (res) => {
            if (res.data.status === 200) {
              this.dishes = res.data.data.map(d => ({
                id: d.id,
                name: d.name,
                category: this.mapCategory(d.category),
                image: d.imageUrl // 注意这里的字段名需与你后端返回的一致
              }))
            }
          },
          fail: () => {
            uni.showToast({ title: '获取菜单失败', icon: 'none' })
          }
        })
      },
      mapCategory(code) {
        const map = {
          '1': '主食',
          '2': '饮品'
        }
        return map[code] || '其他'
      }
    },
    onLoad() {
      this.fetchDishes()
    }
  }

  </script>

  <style scoped>
.container {
  padding: 24rpx;
  background-color: #f2f2f7;
  min-height: 100vh;
}

/* 顶部标题 */
.header {
  text-align: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #2c3e50;
  font-family: 'Helvetica Neue', sans-serif;
}

.subtitle {
  font-size: 26rpx;
  color: #7f8c8d;
  margin-top: 8rpx;
}

/* 分类 Tabs */
.tab-bar {
  display: flex;
  justify-content: space-around;
  background-color: #ffffff;
  padding: 12rpx;
  margin-bottom: 24rpx;
  border-radius: 16rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
}

.tab-item {
  padding: 12rpx 32rpx;
  font-size: 28rpx;
  border-radius: 40rpx;
  color: #34495e;
  background-color: #f5f7fa;
  transition: background-color 0.3s, color 0.3s;
}

.tab-item.active {
  background-color: #007aff;
  color: #fff;
  font-weight: bold;
}

/* 菜品列表 */
.dish-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.dish-card {
  width: 48%;
  background-color: #ffffff;
  margin-bottom: 24rpx;
  border-radius: 24rpx;
  overflow: hidden;
  box-shadow: 0 6rpx 16rpx rgba(0, 0, 0, 0.06);
  transition: transform 0.3s;
}

.dish-card:hover {
  transform: translateY(-4rpx);
}

/* 图片自适应显示长图 */
.dish-img {
  width: 100%;
  height: 360rpx;
  object-fit: cover;
  border-bottom: 1rpx solid #eee;
}

/* 菜品信息 */
.dish-info {
  padding: 20rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.dish-name {
  font-size: 30rpx;
  font-weight: bold;
  color: #2d3436;
  text-align: center;
}

/* 按钮样式 */
.add-btn {
  margin-top: 20rpx;
  width: 100%;
  border-radius: 16rpx;
  background-color: #007aff;
  color: #fff;
  font-size: 26rpx;
  transition: background-color 0.3s;
}

.add-btn:hover {
  background-color: #005bb5;
}

/* 购物车按钮 */
.floating-cart-btn {
  position: fixed;
  bottom: 80rpx;
  right: 40rpx;
  width: 100rpx;
  height: 100rpx;
  background-color: #ff6f61;
  color: #fff;
  font-size: 44rpx;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 6rpx 16rpx rgba(0, 0, 0, 0.25);
  z-index: 999;
}
</style>