<template>
	<my-nav-bar :title="'我的'"></my-nav-bar>
		<view class="container-box">
			<view class="user-box">
				<view class="user-image-box">
					<image :src="userInfo.avatarUrl || '/static/my/用户头像.png'" class="user-image"></image>
				</view>
				<view class="user-data">
					<text class="user-data-text">账号名称：{{userInfo.nickName || '未登录'}}</text>
					<text class="user-data-text">账号ID：{{userInfo.openid || '无'}}</text>
					<text class="user-data-text">收货地址：{{userInfo.address || '请设置'}}</text>
				</view>
			</view>
			<view class="grid-container">
				<view class="grid-row">
					<view v-for="item in orderItems" class="grid-item">
						<image :src="item.icon" class="icon"></image>
						<text class="grid-text">{{item.name}}</text>
					</view>
				</view>
				<view class="grid-row">
					<view v-for="item in settingItems" class="grid-item">
						<image :src="item.icon" class="icon"></image>
						<text class="grid-text">{{item.name}}</text>
					</view>
				</view>
			</view>
			<view class="about-box">
				<view class="about-content-box">
					<text class="about-text">意见投递</text>
				</view>
				<view class="about-content-box">
					<text class="about-text">关于我们</text>
				</view>
				<view class="about-content-box">
					<text class="about-text">打赏</text>
				</view>
			</view>
		</view>
	<my-tab-bar></my-tab-bar>
</template>

<script>
	export default {
	  data() {
	    return {
		  showLoginPopup: false,
		  userInfo: {}, // 用户信息
	      orderItems: [
	        {name: '待付款', icon: '/static/my/待付款.png'},
	        {name: '待收货', icon: '/static/my/待收货.png'},
			{name: '待评价', icon: '/static/my/待评价.png'},
			{name: '退换/售后', icon: '/static/my/退换.png'},
			{name: '全部订单', icon: '/static/my/订单.png'}
	      ],
	      settingItems: [
	        {name: '客户服务', icon: '/static/my/客服中心.png'},
			{name: '消息中心', icon: '/static/my/消息中心.png'},
			{name: '商品收藏', icon: '/static/my/商品收藏.png'},
			{name: '我的足迹', icon: '/static/my/我的足迹.png'},
			{name: '设置', icon: '/static/my/设置.png'}
	      ]
	    }
	  }
	}
</script>

<style scoped>
	.container-box{
		min-height: 100vh;
		background-color: #ebebeb;
		padding-top: 20rpx;
	}
	
	.user-box{
		position: relative;
		flex: 1;
		height: 200rpx;
		background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
		border-radius: 40rpx;
		margin: 0 14rpx;
		align-items: center;
		display: flex;
	}
	
	.user-image-box{
		height: 120rpx;
		width: 120rpx;
		margin: 30rpx;
		background-color: #fff;
		border-radius: 50rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	
	.user-image{
		width: 85%;
		height: 85%;
	}
	
	.user-data{
		display: flex;
		justify-content: center;
		align-items: left;
		flex-direction: column;
		margin-left: 30rpx;
	}
	
	.user-data-text{
		font-size: 30rpx;
		margin: 5rpx 0;
	}
	
	.grid-container {
	  padding: 10rpx 0;
	}
	
	.grid-row {
	  display: flex;
	  justify-content: space-between;
	  align-items: center;
	  height: 150rpx;
	  margin: 15rpx;
	  margin-bottom: 0rpx;
	  padding:10rpx 5rpx;
	  background-color: #fff;
	  border: #fff 1rpx;
	  border-radius: 30rpx;
	}
	
	.grid-item {
	  flex: 1;
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	  min-width: 0; /* 防止内容溢出 */
	}
	
	.grid-text{
		font-size: 30rpx;
		font-weight: 500;
	}
	
	.icon{
		width: 60rpx;
		height: 60rpx;
	}
	
	.about-box{
		display: flex;
		flex-direction: column;
		flex: 1;
		background-color: #fff;
		margin: 10rpx 14rpx;
	}
	
	.about-content-box{
		height: 100rpx;
		border-top: #d3d3d3 solid 1rpx;
		display: flex;
		align-items: center;
		padding: 20rpx;
	}
	
	.about-text{
		font-size: 32rpx;
		font-weight: 600;
		color: #7f7f7f;
	}
</style>