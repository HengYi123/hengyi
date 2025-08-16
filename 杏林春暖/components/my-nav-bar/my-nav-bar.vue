<template>
	<view class="custom-nav-bar" :style="{ paddingTop: statusBarHeight + 'px'}">
		<view class="nav-content">
		    <text class="title">{{title}}</text>
		</view>
	</view>
	<view class="nav-placeholder" :style="{ height: navTotalHeight + 'px' }"></view>
</template>

<script>
	export default{
		props: {
		    title: {
		      type: String,
		      default: '中药材大全' // 默认值
		    }
		},
		data(){
			return{
				statusBarHeight:0,
				navTotalHeight:0
			}
		},
		mounted(){
			const systemInfo = uni.getSystemInfoSync();
			this.statusBarHeight = systemInfo.statusBarHeight;
			this.navTotalHeight = this.statusBarHeight + 44;
			
			// 将高度信息传递给父组件
			this.$emit('height-change', this.navTotalHeight);
		}
	}
</script>

<style scoped>
	.custom-nav-bar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 1000;
		background-color: #F8F8F8;
	}
	
	.nav-placeholder {
	  width: 100%;
	  background-color: transparent; /* 透明背景 */
	}
	
	.nav-content {
		height: 44px;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>