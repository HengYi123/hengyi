<template>
	<view class="content-page">
		<view class="back" @click="goBack">
			<image src="/static/tabbar/返回.png" class="back-image"></image>
		</view>
		<view v-if="loading" class="loading">数据加载中...
		</view>
		<view v-else class="data-container">	
			<image 
				class="search-image"
				:src=getImage()
				mode="widthFix"
				lazy-load="true"
				>
			</image>
			<view class="search-keyword">{{keyword}}</view>	
			<view class="search-keyword-price">
				<text class="label">价格:</text>10.9元/克
			</view>
			<view v-if="flavor" class="data-item">
				<text class="label">味道:</text> {{ flavor }}
			</view>
			<view v-if="effect" class="data-item">
				<text class="label">功效:</text> {{ effect }}
			</view>
			<view v-if="application" class="data-item">
				<text class="label">应用:</text> {{ application }}
			</view>
			<view v-if="pharmacology" class="data-item">
				<text class="label">药理:</text> {{ pharmacology }}
			</view>
			<view v-if="contraindication" class="data-item">
				<text class="label">禁忌:</text> {{ contraindication }}
			</view>
			<view v-if="error" class="error-message">
					数据获取失败: {{ error }}
			</view>
		</view>
		<view class="buttom-tabbar">
			<view class="more">
				<image src="/static/tabbar/店铺.png" class="image-item"></image>
				<image src="/static/tabbar/客服.png" class="image-item"></image>
				<image :src="cartClicked ? '/static/tabbar/购物车-点击后.png' : '/static/tabbar/购物车-点击前.png'"
					class="image-item" 
					@click="goToCart">
				</image>
			</view>
			<view class="shopBox">
				<view 
					class="addToCart"
					@click="add"
					>
					<image src="/static/tabbar/添加购物车.png" class="add-image"></image>
				</view>
				<view class="shoppingCart">立即购买</view>
			</view>
		</view>
	</view>
</template>

<script>
	const apiCache = {};
	export default{
		data(){
			return{
				keyword:'',		//接受的搜索关键字
				loading:true,	//加载状态
				error: '',      // 错误信息
				flavor:'',
				effect: '',
				application: '',
				pharmacology: '',
				contraindication:'',
				cartClicked: false, // 购物车图标点击状态
				cartAdded: false,   // 添加购物车按钮状态
			};
		},
		onLoad(options){
			//接收传递的名字
			this.keyword = decodeURIComponent(options.keyword);
			this.keyword = options.keyword ? decodeURIComponent(options.keyword) : '';
			//记录来源页面标识
			this.sourcePage = options.source;
			this.loadMedicinesData();
		},
		methods:{
			goBack(){
				if (this.sourcePage === 'cart') {
				  // 从购物车跳转来时，正常返回
				  uni.navigateBack({ delta: 1 });
				} else {
				  // 其他来源（如主页）使用重定向
				  uni.redirectTo({ url: '/pages/index/index' });
				}
			},
			
			goToCart(){
				this.cartClicked = true;
				uni.navigateTo({
					url: '/pages/shoppingCart/shoppingCart'
				})
			},
			
			getImage(){
				const iamgeData = `/static/herb/${this.keyword}.jpg`;
				return	iamgeData
			},
			
			async loadMedicinesData() {
				try {
					this.loading = true;
					this.error = '';
					
					// 检查缓存
					if (apiCache[this.keyword]) {
						this.extractFields(apiCache[this.keyword]);
						return;
					}
					
					// 发起请求
					const getUrl = `http://api.zhyunxi.com/api.php?api=87&key=d51b5be2f98dd9bdf06e0bb47dea87fc&name=${encodeURIComponent(this.keyword)}`;
					
					const [res, err] = await new Promise(resolve => {
					    uni.request({
					        url: getUrl,
					        timeout: 10000,
					        dataType: 'json',
					        success: (res) => resolve([res, null]),
					        fail: (err) => resolve([null, err])
					    });
					});
					console.log(getUrl)
					
					// 错误处理
					if (err || !res || res.statusCode !== 200) {
					      throw new Error(err?.message || `请求失败，状态码：${res?.statusCode}`);
					}
					
					// 增加业务状态码检查
					if (res.data.code !== 0) {
					  throw new Error(res.data.msg || 'API返回业务错误');
					}
					
					// 检查数据有效性
					if (!res.data.data || res.data.data.length === 0) {
					  throw new Error(`未找到"${this.keyword}"的相关数据`);
					}
					
					// 缓存数据
					apiCache[this.keyword] = res.data;
					
					// 提取所需字段
					this.extractFields(res.data);
					
				} catch (err) {
					this.error = err.message;
				} finally {
					this.loading = false;
				}
			},
			
			// 提取特定字段
			extractFields(data) {
				// 检查数据结构
				if (!data?.data || !Array.isArray(data.data)) {
				    this.error = 'API返回数据结构异常';
				    return;
				}
				// 提取第一个药材数据
				const medicineData = data.data[0] || {};
				// 根据实际API响应结构调整字段映射
				this.flavor = medicineData.flavor || '无相关信息';
				this.effect = medicineData.effect || '无相关信息';
				this.application = medicineData.application || '无相关信息';
				this.pharmacology = medicineData.pharmacology || '无相关信息';
				this.contraindication = medicineData.contraindication || '无相关信息';
			},
			add(){
				const cartItem = {
					id:Date.now(),			//唯一ID
					name:this.keyword,		//商品名称
					price:10.9,				//默认价格
					image:this.getImage(),	//商品图片
					quantity:1				//数量
				}
				
				//获取当前购物车的数据
				let cart = uni.getStorageSync('cart') || [];
				//检查商品是否已经存在
				const existingItem = cart.find(item=>item.name === this.keyword);
				if(existingItem){
					//商品已存在，增加数量
					existingItem.quantity +=1;
				}else{
					//商品不存在，添加到购物车
					cart.push(cartItem);
				}
				//保存更新后的购物车数据
				uni.setStorageSync('cart',cart);
				//显示添加成功提示
				uni.showToast({
					title:'已添加到购物车',
					icon:'success',
					duration:1500
				});
				//更新全局购物车的状态
				uni.$emit('cart-updated',cart);
			}
		}
	}
</script>

<style scoped>
	.content-page{
		background-color: #ebebeb;
		min-height: 100vh;
	}
	
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
	
	.search-keyword {
	  font-size: 45rpx;
	  font-weight: bold;
	  padding: 10rpx;
	}
	
	.search-image{
		width: 100%;
		height: 100%;
	}
	
	.data-item{
		padding: 10rpx;
		font-size: 36rpx;
	}
	
	.data-item,.search-keyword-price{
		border-top: #c0c0c0 dashed 2rpx;
	}
	
	.label{
		font-weight: bold;
	}
	
	.search-keyword-price{
		padding: 10rpx;
		font-size: 36rpx;
	}
	
	.buttom-tabbar{
		width: 100%;
		height: 100rpx;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		position: fixed;
		background-color: rgba(255, 255, 255);
		padding-bottom: env(safe-area-inset-bottom); /* 适配全面屏手机 */
		bottom: 5rpx; 
		left: 0;
		right: 0;
		z-index: 999;
	}
	
	.more{
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
		width: 200rpx;
		margin-right: 20rpx;
	}
	
	.addToCart{
		width: 120rpx;
		text-align: center;
		background-color: #f1d94d;
	}
	
	.shoppingCart{
		flex:1;
		height: 80rpx;
		background-color: #dd4949;
		font-size: 50rpx;
		color: #e8f5e9;
		text-align: center;
	}
	
	.image-item{
		width: 50rpx;
		height: 50rpx;
	}
	
	.add-image{
		width: 70rpx;
		height: 70rpx;
	}
	
	.shopBox{
		border-radius: 20rpx;
		display: flex;
		flex-direction: row;
		justify-content: center;
		width: 500rpx;
	}
</style>