<template>
	<!-- 返回按钮 -->
	<view class="back" @click="goBack">
		<image src="/static/tabbar/返回.png" class="back-image"></image>
	</view>
	<!-- 地图容器 -->
	<map 
		id="myMap"
		class="map-container"
		:latitude="latitude"
		:longitude="longitude"
		:markers="markers"
		:show-location="true"
		:controls="controls"
		@controltap="onControlTap"
	></map>
	
	<!-- 定位按钮 -->
	<view class="locate-btn" @click="startLocation">
		<image src="/static/map/location.png"></image>
	</view>
	
	<!-- 地址显示栏 -->
	<view class="address-bar">
		<image src="/static/map/location-pin.png" class="address-icon"></image>
		<text class="address-text">{{ currentAddress || '定位中...' }}</text>
	</view>
	
	<!-- 地图水印 -->
	<view class="map-watermark">
		<text>高德地图</text>
		<text>©2025 AMap · GS 撖 (2023)1171号</text>
	</view>
</template>

<script>	
	export default{
		data(){
			return {
				latitude: 39.90923, 					// 默认北京坐标
				longitude: 116.397428,					// 初始中心经度
				currentAddress: '',						// 当前地址文本
				markers: [{								// 标记点配置
					id: 1,
					latitude: 39.90923,
					longitude: 116.397428,
					iconPath: '/static/map/pin.png',	// 标记图标
					width: 30,			
					height: 40,
					callout: {							// 标记点气泡
						content: '当前位置',
						color: '#fff',
						bgColor: '#1A73E8',
						padding: 8,
						borderRadius: 4,
						display: 'ALWAYS'				// 常显气泡
					}
				}],
				controls: [],							// 地图控件数组
				sourcePage:''							// 页面来源标识
			}
		},
		
		onLoad(options){
			// 记录来源页面
			this.sourcePage = options.sourcePage || '';	
			// 页面加载即定位
			this.checkLocationPermission(); 			
		},
		
		onReady(){
			// 创建地图实例
			this.mapContext = uni.createMapContext('myMap',this);
		},
		methods:{
			// 检查定位权限状态
			checkLocationPermission() {
			    uni.getSetting({
			      success: (res) => {
			        if (res.authSetting['scope.userLocation']) {
			          // 已有权限，直接定位
			          this.doLocation();
			        } else {
			          // 无权限，显示提示但不强制请求
			          this.currentAddress = '请点击定位按钮获取位置';
			        }
			      }
			    });
			},
			
			// 触发定位流程
			startLocation() {
			  // 先检查并请求定位权限
			  uni.authorize({
			    scope: 'scope.userLocation',
			    success: () => {
			      // 权限已授予，开始定位
			      this.doLocation();
			    },
			    fail: () => {
			      uni.hideLoading();
			      uni.showModal({
			        title: '提示',
			        content: '需要位置权限才能获取当前位置',
			        confirmText: '去设置',
			        success: (res) => {
			          if (res.confirm) {
			            // 引导用户打开设置
			            uni.openSetting();
			          }
			        }
			      });
			    }
			  });
			},
			
			// 实时定位核心方法
			doLocation() {
				uni.showLoading({
					title: '定位中...',
					mask: true
				});
				uni.getLocation({
					type: 'gcj02', 		// 高德坐标系
					altitude: true,	 	// 获取高度信息
					success: (res) => {
						// 更新坐标数据
						this.latitude = res.latitude;
						this.longitude = res.longitude;
						
						// 更新标记点
						this.markers[0] = {
							...this.markers[0],
							latitude: res.latitude,
							longitude: res.longitude
						};
						
						// 坐标转地址（逆地理编码）
						this.reverseGeocode(res.latitude, res.longitude);
						
						// 移动地图视角到当前位置
						this.mapContext.moveToLocation({
							latitude: res.latitude,
							longitude: res.longitude,
							success: () => {
								uni.hideLoading();
							}
						});
					},
					fail: (err) => {
						uni.hideLoading();
						uni.showToast({ 
							title: '定位失败，请检查权限设置', 
							icon: 'none',
							duration: 3000
						});
					}
				});
			},
			
			// 逆地理编码
			async reverseGeocode(lat, lng) {
				try {
					const res = await uni.request({
						url: `https://restapi.amap.com/v3/geocode/regeo?key=e6ad274c3e0a6139d29e493e3c689bed&location=${lng},${lat}`
					});
					if (res[1].data.status === '1') {
						this.currentAddress = res[1].data.regeocode.formatted_address;
					} else {
						this.currentAddress = '地址解析失败';
					}
				} catch (e) {
					this.currentAddress = '地址解析失败';
				}
			},
			
			// 控件点击事件
			onControlTap(e) {
				if (e.controlId === 1) {
					uni.navigateTo({ url: '/pages/map/search' });
				}
			},
			
			// 返回逻辑
			goBack(){
				if (this.sourcePage === 'cart') {
				  // 从购物车跳转来时，正常返回
				  uni.navigateBack({ delta: 1 });
				} else {
				  // 其他来源（如主页）使用重定向
				  uni.redirectTo({ url: '/pages/index/index' });
				}
			},
		}
	}
	
</script>

<style>
	.back{
		display: flex;
		position: fixed;
		top: 100rpx;
		left: 20rpx;
		z-index: 1000;
		background-color: #f4f4f4;
		border-radius: 50%;
		width: 60rpx;
		height: 60rpx;
		justify-content: center;
		align-items: center;
		box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.1);
	}
	
	.back-image{
		width: 40rpx;
		height: 40rpx;
	}
	
	.back:active{
		transform: scale(0.95);
		background-color: rgba(240,240,240,0.9);
	}
	
	.map-container {
		width: 100%;
		height: 100vh;
		position: fixed;
		top: 0;
		left: 0;
	}
	
	.locate-btn {
		position: fixed;
		right: 30rpx;
		bottom: 250rpx;
		width: 90rpx;
		height: 90rpx;
		background: white;
		border-radius: 50%;
		box-shadow: 0 0 10px rgba(0,0,0,0.2);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 100;
	}
	
	.locate-icon {
		width: 50rpx;
		height: 50rpx;
	}
	
	.address-bar {
		position: fixed;
		bottom: 140rpx;
		left: 30rpx;
		right: 30rpx;
		background: rgba(255, 255, 255, 0.9);
		padding: 25rpx 30rpx;
		border-radius: 16rpx;
		box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1);
		display: flex;
		align-items: center;
		z-index: 100;
	}
	
	.address-icon {
		width: 40rpx;
		height: 40rpx;
		margin-right: 15rpx;
	}
	
	.address-text {
		font-size: 32rpx;
		color: #333;
		flex: 1;
	}
	
	.map-watermark {
		position: fixed;
		bottom: 30rpx;
		left: 0;
		right: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		z-index: 50;
	}
	
	.map-watermark text {
		font-size: 24rpx;
		color: rgba(0, 0, 0, 0.5);
		line-height: 1.5;
	}
</style>