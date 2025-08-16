<template>
	<view 
		class="back-to-top-btn"
		:class="[{ 'is-visible': showBackToTop }]"
		v-show="showBackToTop" 
		@click="scrollToTop">
		<view class="image-box">
			<image 
			src="/static/置顶.png" 
			mode="aspectFit" 
			class="arrow-icon"/>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			showBackToTop: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			scrollToTop() {
				this.$emit('back-to-top'); // 通知父组件
			}
		},
		watch: {
			// 监听props变化并输出日志
			showBackToTop(newVal) {
				console.log("按钮显示状态:", newVal);
			}
		}
	};
</script>

<style scoped>
	.back-to-top-btn {
		position: fixed;
		right: 30rpx;
		bottom: 180rpx;
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		background-color: #f8f4e5;
		box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.15);
		z-index: 400;
		transition: transform 0.3s ease, opacity 0.3s ease;
		display: flex;
		justify-content: center;
		align-items: center;
		opacity: 0.9;
	}
	
	.back-to-top-btn.is-visible {
		opacity: 0.8;
		transform: translateY(0);
	}
	
	.arrow-icon {
		width: 50rpx;  
		height: 50rpx; 
	}
</style>