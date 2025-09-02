<template>
	<view class="herb-section-title">
		<view class="decoration-line-top"></view>
		<text class="title-text">药材专区</text>
		<view class="decoration-line-button"></view>
	</view>
</template>

<script>
</script>

<style scoped>
	.herb-section-title{
		position:relative;
		font-size: 25rpx;
		background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
		border-radius: 16rpx;
		box-shadow: 0 8rpx 25rpx rgba(76, 175, 80, 0.15);
		overflow: hidden;
	}
		
	.title-text {
		font-size: 36rpx;
		font-weight: bold;
		color: #2E7D32; 
		text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
		letter-spacing: 2rpx;
		position: sticky; /* 粘性定位（滚动时固定顶部）*/
		top: 0;
		padding-left: 20rpx;    
	}
	
	.decoration-line-top {
		height: 4rpx;
		background: linear-gradient(90deg, #4CAF50, #81C784, transparent);
		border-radius: 2rpx;
	}
	
	.decoration-line-button {
		height: 4rpx;
		background: linear-gradient(90deg, #4CAF50, #81C784, transparent);
		margin-top: 10rpx;
		border-radius: 2rpx;
	}
</style>