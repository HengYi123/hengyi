<template>
	<view class="carousel-container">
		<!-- 循环播放模式 -->
		<swiper 
			circular="true"
			indicator-dots="true" 
			indicator-color="#fff" 
			indicator-active-color="#007aff" 
			autoplay="true" 
			interval="3000"
			>
			<swiper-item>
				<view class="image-wrapper">
					<image src="/static/carousel/carousel-01.jpg" mode="widthFix" style="width: 100%;height: 100%;"></image>
				</view>
			</swiper-item>
			<swiper-item>
				<view class="image-wrapper">
					<image src="/static/carousel/carousel-02.jpg" mode="widthFix" style="width: 100%;height: 100%;"></image>
				</view>
			</swiper-item>
			<swiper-item>
				<view class="image-wrapper">
					<image src="/static/carousel/carousel-03.jpg" mode="widthFix" style="width: 100%;height: 100%;"></image>
				</view>
			</swiper-item>
		</swiper>
	</view>

</template>

<script>
</script>

<style scoped>
	.image-wrapper {
	  height: 300rpx !important; 
	}
	
	swiper {
	  height: 300rpx !important;     /* 指定 swiper 高度 */
	}
	
	.carousel-container{
		width:100%;
		min-height: 300rpx; 
		margin: 0 auto;	/* 水平居中 */
		margin-bottom: 10rpx;
	}
</style>