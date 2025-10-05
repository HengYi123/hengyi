<template>
	<my-nav-bar :title="'我的'"></my-nav-bar>
		<view class="container-box">
			<!-- 用户信息区域 -->
			<view class="user-box">
				<!-- 用户头像（点击触发授权弹窗） -->
				<view class="user-image-box">
					<image 
						:src="userInfo.avatarUrl || '/static/my/用户头像.png'" 
						class="user-image"
						@click="showAuthPopup = true">
					</image>
				</view>
				<!-- 用户数据展示 -->
				<view class="user-data">
					<text class="user-data-text">账号名称：{{userInfo.nickName || '未登录'}}</text>
					<text class="user-data-text">账号ID：{{userInfo.openid || '无'}}</text>
					<!-- 收货地址（点击跳转地图页） -->
					<view class="address-row" @click="navigateToMap">
						<text class="user-data-text text-color">收货地址：{{userInfo.address || '请设置'}}</text>
						<image 
							src="/static/my/右箭头.png" 
							class="location-icon">
						</image>
					</view>
				</view>
			</view>
			<!-- 功能入口网格布局 -->
			<view class="grid-container">
				<!-- 订单管理入口 -->
				<view class="grid-row">
					<view v-for="item in orderItems" class="grid-item">
						<image :src="item.icon" class="icon"></image>
						<text class="grid-text">{{item.name}}</text>
					</view>
				</view>
				<!-- 设置管理入口 -->
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
				<view class="about-content-box" @click="toZanZhu">
					<text class="about-text text-color">打赏</text>
				</view>
			</view>
			<!-- 授权登录弹窗（条件渲染） -->
			<view v-if="showAuthPopup" class="popup-container">
				<!-- 遮罩层 -->
				<view class="popup-mask" @tap="showAuthPopup = false"></view>
				<!-- 弹窗内容 -->
				<view class="popup-content">
					<view class="popup-header">
						<text class="popup-title">微信授权登录</text>
					</view>
					<!-- 头像设置区域 -->
					<view class="avatar-section">
						<view class="avatar-preview">
							<image :src="tempAvatar || '/static/my/用户头像.png'" class="preview-image"></image>
						</view>
						<button class="avatar-btn" @tap="selectAvatar">选择头像</button>
					</view>
					<!-- 昵称输入区域 -->
					<view class="nickname-section">
						<input 
							type="nickname"
							v-model="tempNickname"
							placeholder="设置昵称"
							class="nickname-input"
							placeholder-style="color: #999;"
							@blur="saveNickname" />
					</view>
					<!-- 提示信息 -->
					<view class="auth-tip">
						<text>申请获取您的公开信息（昵称、头像等）</text>
					</view>
					<!-- 按钮区域 -->
					<view class="btn-group">
						<button class="cancel-btn" @tap="showAuthPopup = false">取消</button>
						<button class="confirm-btn" @tap="confirmAuth">同意</button>
					</view>
				</view>
			</view>
			<!-- 底部提示栏 -->
			<view class="prompt-box">
				<image src="/static/my/提醒.png" class="prompt-iamge"></image>
				<text>点击头像进行登录</text>
			</view>
		</view>
	<my-tab-bar></my-tab-bar>
</template>

<script>
	export default {
		data() {
			return {
				showAuthPopup: false,		// 控制授权弹窗显示
				tempAvatar: '',          	// 临时存储选择的头像路径
				tempNickname: '',          	// 临时存储输入的昵称
				userInfo: {
					address: uni.getStorageSync('selectedAddress') || '', // 从本地存储读取地址
				}, 			 	// 用户信息对象
				// 订单管理入口配置
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
		},
		
		onLoad() {
			// 添加全局事件监听器
			uni.$on('updateUserAddress', this.updateAddress);
			// 页面加载时检查本地存储中是否有地址
			const savedAddress = uni.getStorageSync('selectedAddress');
			if (savedAddress) {
				this.userInfo.address = savedAddress;
			}
		},
		
		onUnload() {
			// 移除事件监听器
			uni.$off('updateUserAddress', this.updateAddress);
		},
		
		methods:{
			// 跳转至地图页面
			navigateToMap() {
				uni.navigateTo({
					url:'/pages/map/map?source=my&address=' + encodeURIComponent(this.userInfo.address || '')
				});
			},

			toZanZhu(){
				uni.navigateTo({
					url:'/pages/my/zanZhu'
				});
			},
			
			updateAddress(newAddress) {
				this.userInfo.address = newAddress;
				// 保存到本地存储
				uni.setStorageSync('selectedAddress', newAddress);
			},
			  
			// 选择头像
			selectAvatar() {
				uni.chooseImage({
					count: 1, // 最多选择1张图片
					sizeType: ['compressed'], // 压缩图
					sourceType: ['album', 'camera'], // 从相册和相机选择
					success: (res) => {
						// 获取选择的图片临时路径
						const tempFilePaths = res.tempFilePaths;
						if (tempFilePaths.length > 0) {
							// 更新临时头像
							this.tempAvatar = tempFilePaths[0];
							uni.showToast({
								title: '头像已选择',
								icon: 'success'
							});
						}
					},
					fail: (err) => {
						console.error('选择头像失败', err);
						uni.showToast({
							title: '选择头像失败',
							icon: 'none'
						});
					}
				});
			},
			  
			// 保存昵称
			saveNickname(e) {
				this.tempNickname = e.detail.value;
			},
			  
			// 确认授权
			confirmAuth() {
				if (!this.tempNickname) {
					  uni.showToast({
						title: '请设置昵称',
						icon: 'none'
					  });
					  return;
				}
				  
				// 保存用户信息
				this.userInfo = {
					avatarUrl: this.tempAvatar || '/static/my/用户头像.png',
					nickName: this.tempNickname,
					openid: 'wx_' + Date.now().toString().slice(-6) // 模拟openid
				};
				  
				this.showAuthPopup = false;
				uni.showToast({
					title: '授权成功',
					icon: 'success'
				});
			}
		}
	}
</script>

<style scoped>
	.container-box{
		min-height: 100vh;
		background-color: #ebebeb;
		padding-top: 20rpx;
		position: relative;
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
	
	.address-row {
		display: flex;
		align-items: center;
		flex-direction: row;
	}
	
	.address-row .user-data-text {
	    white-space: nowrap; /* 禁止换行 */
	    overflow: hidden; /* 隐藏溢出内容 */
	    text-overflow: ellipsis; /* 超出部分显示省略号 */
	    max-width: 450rpx;
	}
	
	.location-icon {
		width: 26rpx; 
		height: 26rpx; 
		margin-left: 5rpx;
		position: relative;
		top: 3rpx;
	}
	
	.popup-container {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		z-index: 1000;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
	}
	
	.popup-mask {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.5);
		z-index: 1001;
	}
	
	.popup-content {
		position: relative;
		width: 100%;
		background: #fff;
		border-radius: 30rpx 30rpx 0 0;
		padding: 30rpx;
		z-index: 1002;
		box-sizing: border-box;
		animation: popupSlide 0.35s ease-out; /* 添加入场动画 */
	}
	
	@keyframes popupSlide {
		from { transform: translateY(100%); }
		to { transform: translateY(0); }
	}
	
	.popup-header {
		padding-bottom: 20rpx;
		text-align: center;
		border-bottom: 1rpx solid #f0f0f0;
	}
	
	.popup-title {
		font-size: 34rpx;
		font-weight: bold;
		color: #333;
		letter-spacing: 1rpx;
	}
	
	.avatar-section {
		padding: 20rpx 0;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.avatar-preview {
		width: 140rpx;
		height: 140rpx;
		border-radius: 50%;
		overflow: hidden;
		border: 2rpx solid #f0f0f0;
		margin-bottom: 15rpx;
		background: #f8f8f8;
	}
	
	.preview-image {
		width: 100%;
		height: 100%;
	}
	
	.avatar-btn {
		font-size: 28rpx;
		color: #07c160;
		background: none;
		padding: 8rpx 30rpx;
		border: 1rpx solid #07c160;
		border-radius: 40rpx;
		line-height: 1.4;
	}
	
	.nickname-section {
		padding-bottom: 25rpx;
	}
	
	.nickname-input {
		width: 100%;
		height: 80rpx;
		border: 1rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 20rpx;
		font-size: 30rpx;
		background: #f9f9f9;
	}
	
	.auth-tip {
		padding: 18rpx 0;
		background: #f8f8f8;
		text-align: center;
		font-size: 26rpx;
		color: #777;
		border-radius: 12rpx;
		margin: 20rpx 0;
	}
	
	.btn-group {
		display: flex;
		padding-top: 15rpx;
		gap: 15rpx;
	}
	
	.cancel-btn, .confirm-btn {
		flex: 1;
		height: 85rpx;
		font-size: 32rpx;
		border-radius: 45rpx;
	}
	
	.cancel-btn {
		color: #666;
		background: #f0f0f0;
		border: none;
	}
	
	.confirm-btn {
		color: #fff;
		background: #07c160;
		font-weight: bold;
		border: none;
	}
	
	.prompt-box{
		position: fixed;
		bottom: 170rpx;
		left: 0;
		right: 0;
		display: flex;
		align-items: center;
		height: 75rpx;
		background: rgba(240, 212, 127, 0.95);
		padding: 0 25rpx;
		z-index: 100;
		color: #5a471c;
		font-size: 28rpx;
		font-weight: 500;
	}
	
	.prompt-iamge{
		width: 40rpx;
		height: 40rpx;
		margin: 0 10rpx;
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
	
	.text-color{
		color:#2E7D32;
	}
</style>