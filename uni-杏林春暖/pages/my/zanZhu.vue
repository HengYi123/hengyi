<template>
  <!-- 使用页面容器包裹所有内容 -->
  <view class="page-container">
    
    <!-- 返回按钮 -->
    <view class="back" @click="goBack">
      <image src="/static/tabbar/返回.png" class="back-image"></image>
    </view>
    
    <!-- 页面标题 -->
    <view class="header">
      <text class="title">支持开发者</text>
      <text class="subtitle">您的支持是我持续创作的动力</text>
    </view>
    
    <!-- 赞助选项 -->
    <view class="donation-options">
      <view 
        v-for="(option, index) in donationOptions" 
        :key="index" 
        class="option-card"
        :class="{ active: selectedOption === index }"
        @click="selectOption(index)"
      >
        <text class="amount">{{ option.amount }}元</text>
        <text class="description">{{ option.description }}</text>
      </view>
    </view>
    
    <!-- 微信支付区域 -->
    <view class="payment-section">
      <view class="section-title">
        <text class="payment-text">自定义</text>
      </view>
      
      <view class="qr-code-container">
        <image src="/static/my/微信收款码.png" class="wxImage"></image>
        <view class="hint">请使用微信扫描二维码完成赞助</view>
      </view>
    </view>
	
  </view>
</template>

<script>
export default {
  data() {
    return {
      selectedOption: null,
      donationOptions: [
        { amount: 5, description: "一杯咖啡" },
        { amount: 10, description: "一份鼓励" },
        { amount: 20, description: "强力支持" },
        { amount: 50, description: "超级赞助" }
      ]
    }
  },
  methods: {
    goBack() {
      uni.navigateBack()
    },
    selectOption(index) {
      this.selectedOption = index
      // 这里可以添加选择金额后的逻辑，比如高亮显示等
    }
  }
}
</script>

<style>
/* 页面容器 */
.page-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 40rpx 30rpx;
}

/* 返回按钮样式 */
.back{
  display: flex;
  position: absolute;
  top: 80rpx;
  left: 30rpx;
  z-index: 1000;
  background-color: #ffffff;
  border-radius: 50%;
  width: 70rpx;
  height: 70rpx;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.back-image{
  width: 40rpx;
  height: 40rpx;
}

.back:active{
  transform: scale(0.95);
  background-color: rgba(240,240,240,0.9);
}

/* 标题样式 */
.header {
  text-align: center;
  margin-top: 60rpx;
  margin-bottom: 50rpx;
}

.title {
  display: block;
  font-size: 46rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
}

.subtitle {
  display: block;
  font-size: 28rpx;
  color: #666;
}

/* 赞助选项样式 */
.donation-options {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 20rpx;
}

.option-card {
  width: 48%;
  background: white;
  border-radius: 16rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
  text-align: center;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.06);
  transition: all 0.3s ease;
  border: 2rpx solid transparent;
}

.option-card.active {
  border-color: #07C160; /* 微信绿色 */
  transform: translateY(-5rpx);
  box-shadow: 0 8rpx 25rpx rgba(0,0,0,0.1);
}

.amount {
  display: block;
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 10rpx;
}

.description {
  display: block;
  font-size: 24rpx;
  color: #888;
}

/* 支付区域样式 */
.payment-section {
  background: white;
  border-radius: 20rpx;
  padding: 40rpx;
  margin-bottom: 40rpx;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.06);
}

.section-title {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 30rpx;
}

.payment-icon {
  width: 50rpx;
  height: 50rpx;
  margin-right: 15rpx;
}

.payment-text {
  font-size: 34rpx;
  font-weight: bold;
  color: #333;
}

.qr-code-container {
  text-align: center;
}

.wxImage {
  width: 400rpx;
  height: 400rpx;
  border-radius: 16rpx;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1);
}

.hint {
  display: block;
  margin-top: 20rpx;
  font-size: 24rpx;
  color: #888;
}

/* 响应式调整 */
@media (max-width: 600px) {
  .option-card {
    width: 100%;
  }
}
</style>