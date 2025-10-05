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
		<image src="/static/map/定位.png" class="locate-image"></image>
	</view>
	
	<!-- 地址显示栏 -->
	<view class="address-bar" @click="openSearchPopup">
		<image src="/static/map/location-pin.png" class="address-icon"></image>
		<text class="address-text">{{ currentAddress || '点击搜索或定位...' }}</text>
	</view>
	
	<!-- 搜索弹窗 -->
	<uni-popup ref="searchPopup" type="dialog">
		<uni-popup-dialog 
			mode="input" 
			title="搜索地点" 
			:value="searchInput"
			placeholder="请输入地址或地名" 
			@confirm="onSearchConfirm"
		></uni-popup-dialog>
	</uni-popup>
	
	<!-- 搜索结果列表 -->
	<view class="search-results" v-if="searchResults.length > 0">
		<scroll-view class="results-scroll" scroll-y>
			<view 
				v-for="(item, index) in searchResults" 
				:key="index" 
				class="result-item"
				@click="selectSearchResult(item)"
			>
				<text class="result-title">{{ item.title }}</text>
				<text class="result-address">{{ item.address }}</text>
			</view>
		</scroll-view>
		<view class="close-results" @click="clearSearchResults">关闭</view>
	</view>
	
	<!-- 地图水印 -->
	<view class="map-watermark">
		<text>腾讯地图</text>
		<text>©2025 Tencent · GS(2023)1171号</text>
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
				controls: [{ 							
					id: 1,
					position: {
						left: 10,
						top: 100,
						width: 40,
						height: 40
					},
					iconPath: '/static/map/定位.png',
					clickable: true
				}],										// 地图控件数组
				sourcePage:'',							// 页面来源标识
				searchInput:'',							// 输入要搜索的值
				searchResults: [], 						// 搜索结果列表
				mapContext: null, 						// 地图上下文
				hasLocationPermission: false
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
			// 控件点击事件 - 添加定位功能
			onControlTap(e) {
				if (e.controlId === 1) {
					// 点击定位控件时触发定位
					this.startLocation();
				}
			},
			
			// 打开搜索弹窗
			openSearchPopup() {
				this.$refs.searchPopup.open();
			},
			
			// 搜索确认
			onSearchConfirm(value) {
				this.searchInput = value;
				this.$refs.searchPopup.close();
				
				if (!this.searchInput.trim()) {
					uni.showToast({ title: '请输入搜索关键词', icon: 'none' });
					return;
				}
				
				// 执行搜索
				this.performSearch(this.searchInput);
			},
			
			// 执行搜索
			async performSearch(keyword) {
				uni.showLoading({ title: '搜索中...', mask: true });
				
				try {
					// 使用腾讯地图地点搜索API[1](@ref)
					const res = await uni.request({
						url: 'https://apis.map.qq.com/ws/place/v1/search',
						data: {
							key: 'OMEBZ-JYNEW-BX6RJ-Y63WC-JS4BE-4SB6C', // 您的腾讯地图key
							keyword: keyword,
							boundary: `region(北京,0)`, // 搜索范围-北京
							page_size: 20 // 返回结果数量
						}
					});
					
					uni.hideLoading();
					
					if (res.data.status === 0) {
						this.searchResults = res.data.data || [];
						
						if (this.searchResults.length === 0) {
							uni.showToast({ title: '未找到相关地点', icon: 'none' });
						}
					} else {
						uni.showToast({ title: `搜索失败: ${res.data.message}`, icon: 'none' });
						console.error('腾讯地图搜索API错误:', res.data);
					}
				} catch (error) {
					uni.hideLoading();
					uni.showToast({ title: '搜索请求异常', icon: 'none' });
					console.error('搜索请求异常:', error);
				}
			},
			
			// 选择搜索结果
			selectSearchResult(item) {
				// 更新地图中心点
				this.latitude = item.location.lat;
				this.longitude = item.location.lng;
				
				// 更新标记点
				this.markers[0] = {
					...this.markers[0],
					latitude: item.location.lat,
					longitude: item.location.lng,
					callout: {
						...this.markers[0].callout,
						content: item.title
					}
				};
				
				// 移动地图视角到选中位置
				this.mapContext.moveToLocation({
					latitude: item.location.lat,
					longitude: item.location.lng
				});
				
				// 创建完整的地址字符串
				const fullAddress = `${item.title}（${item.address}）`;
				
				// 更新地址栏文本
				this.currentAddress = fullAddress;
				
				// 将地址保存到本地存储
				uni.setStorageSync('selectedAddress', fullAddress);
				
				// 将选择的地址传递回 my 页面
				this.returnSelectedAddress(fullAddress, item.location.lat, item.location.lng);
				
				// 清空搜索结果列表
				this.clearSearchResults();
			},
			
			// 返回选择的地址到 my 页面
			returnSelectedAddress(address, lat, lng) {
			    // 获取页面栈
			    const pages = getCurrentPages();
			    const prevPage = pages[pages.length - 2]; // 上一个页面（my页面）
			    
			    if (prevPage && prevPage.route === 'pages/my/my') {
			        // 通过页面实例直接调用方法
			        if (prevPage.$vm && prevPage.$vm.updateAddress) {
			            prevPage.$vm.updateAddress(address);
			        }
			        
			        // 通过全局事件（备用方案）
			        uni.$emit('updateUserAddress', address);
			        
			        // 返回上一页
			        setTimeout(() => {
			            uni.navigateBack({
			                delta: 1,
			                success: () => {
			                    uni.showToast({
			                        title: '地址选择成功',
			                        icon: 'success'
			                    });
			                }
			            });
			        }, 500);
			    } else {
			        // 如果不是从my页面跳转过来，提示用户手动返回
			        uni.showModal({
			            title: '地址已选择',
			            content: `已选择地址：${address}`,
			            confirmText: '确定',
			            showCancel: false
			        });
			    }
			},
			
			// 清空搜索结果
			clearSearchResults() {
				this.searchResults = [];
			},
			
			// 检查定位权限状态
			checkLocationPermission() {
				uni.getSetting({
					success: (res) => {
						if (res.authSetting['scope.userLocation']) {
						  // 已有权限
							this.hasLocationPermission = true;
							this.doLocation(); // 直接定位
						} else {
						  // 无权限
							this.hasLocationPermission = false;
							this.currentAddress = '请点击定位按钮获取位置';
						}
					  },
					fail: () => {
						this.hasLocationPermission = false;
						this.currentAddress = '权限检查失败';
					}
				});
			},
			
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
			    if (this.hasLocationPermission) {
					// 已有权限，直接定位
					this.doLocation();
					return;
			    }
			  
				// 请求定位权限
				uni.authorize({
					scope: 'scope.userLocation',
					success: () => {
					  this.hasLocationPermission = true;
					  this.doLocation();
					},
					fail: (err) => {
						// 用户拒绝授权
						if (err.errMsg.indexOf('deny') !== -1) {
							uni.showModal({
								title: '权限提示',
								content: '需要位置权限才能获取当前位置，请前往设置开启权限',
								confirmText: '去设置',
								success: (res) => {
									if (res.confirm) {
									  // 引导用户打开设置
										uni.openSetting({
											success: (settingRes) => {
												if (settingRes.authSetting['scope.userLocation']) {
													this.hasLocationPermission = true;
													this.doLocation();
												}
											}
										});
									}
								}
							});
						} else {
							uni.showToast({
							  title: '权限请求失败',
							  icon: 'none'
							});
						  }
					}
				});
			},
			
			// 实时定位核心方法
			doLocation() {
			    if (!this.hasLocationPermission) {
					this.startLocation(); // 重新请求权限
					return;
			    }
			  
			    uni.showLoading({
					title: '定位中...',
					mask: true
			    });
			  
			    uni.getLocation({
					type: 'gcj02', // 高德坐标系
					altitude: true, // 获取高度信息
					success: (res) => {
					    // 更新坐标数据
						this.latitude = res.latitude;
						this.longitude = res.longitude;
					  
						// 更新标记点
						this.markers = [{
							...this.markers[0],
							latitude: res.latitude,
							longitude: res.longitude
						}];
					  
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
					  
						// 处理不同错误类型
						let errorMsg = '定位失败';
						if (err.errMsg.indexOf('auth') !== -1) {
							errorMsg = '定位权限未开启';
							this.hasLocationPermission = false;
						} else if (err.errMsg.indexOf('available') !== -1) {
							errorMsg = '定位服务不可用';
						}
					  
					    uni.showToast({ 
							title: errorMsg, 
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
						url: 'https://apis.map.qq.com/ws/geocoder/v1/',
						data: {
						    key: 'OMEBZ-JYNEW-BX6RJ-Y63WC-JS4BE-4SB6C', // 使用你提供的密钥
						    location: `${lat},${lng}`
						}
					});
					
					if (res && res.data && res.data.status === 0) {
						// 腾讯地图API成功响应
						this.currentAddress = res.data.result.address || '地址解析成功但无详细地址';
					} else {
						// 腾讯地图API返回错误
						this.currentAddress = '地址解析失败';
						console.error('腾讯地图逆地理编码失败:', res ? res.data : error);
					}
				} catch (e) {
					this.currentAddress = '地址解析失败';
					console.error('逆地理编码请求异常:', e);
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
	
	.locate-image {
		width: 50rpx;
		height: 50rpx;
		z-index: 400;
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
	
	.search-results {
		position: fixed;
		bottom: 140rpx;
		left: 30rpx;
		right: 30rpx;
		background: white;
		border-radius: 16rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.15);
		z-index: 1000;
		max-height: 60vh;
		display: flex;
		flex-direction: column;
	}
	
	.results-scroll {
		flex: 1;
		max-height: calc(60vh - 80rpx);
	}
	
	.result-item {
		padding: 25rpx 30rpx;
		border-bottom: 1rpx solid #f0f0f0;
	}
	
	.result-item:active {
		background-color: #f9f9f9;
	}
	
	.result-title {
		display: block;
		font-size: 32rpx;
		color: #333;
		margin-bottom: 8rpx;
		font-weight: bold;
	}
	
	.result-address {
		display: block;
		font-size: 26rpx;
		color: #888;
	}
	
	.close-results {
		height: 80rpx;
		line-height: 80rpx;
		text-align: center;
		font-size: 32rpx;
		color: #1A73E8;
		border-top: 1rpx solid #f0f0f0;
		font-weight: bold;
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