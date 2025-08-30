<template>
	<!-- 根据购物车是否为空动态添加fixed-layout类 -->
	<view :class="{'fixed-layout': cartItems.length === 0}">
		<my-nav-bar :title="'我的购物车'"></my-nav-bar>
			<!-- 购物车头部区域 -->
			<view class="cart-header">
				<text class="cart-title">购物清单</text>
				<view class="cart-content">
					<!-- 全选按钮：点击切换全选状态 -->
					<view class="selected-all" @click.stop="toggleSelectAll">
						<image 
							class="selector-all-image" 
							:src="isAllSelected ? '/static/选中.png' : '/static/未选中.png'"
						></image>
						<text class="select-all-text">全选</text>
					</view>
					<!-- 删除按钮：点击删除选中商品 -->
					<view class="delete-box" @click="deleteSelectedItems">
					<image 
						class="trash-icon" 
						src="/static/删除.png"
					></image>
				</view>
				<text class="cart-count">总共({{cartItems.length}}条)</text>
				</view>
			</view>
			<!-- 空购物车提示 -->
			<view v-if="cartItems.length === 0" class="empty-cart">
				<image src="/static/购物车.png" class="empty-image"></image>
				<text class="empty-text">购物车空空如也</text>
				<text class="empty-tip">快去选购你心仪的商品吧~</text>
				<button class="go-shopping-button" @click="goToIndex">去逛逛</button>
			</view>
			<!-- 购物车商品列表 -->
			<scroll-view class="cart-container" scroll-y>
				<view 
					v-for="(item, index) in cartItems" 
					:key="item.id" 
					class="cart-item"
					@click="goToContentPage(item)"
				>
					<!-- 单选按钮：点击切换选中状态 -->
					<view class="selector-container" @click.stop="toggleSelectItem(index)"
						scroll-y :style="cartItems.length === 0 ? 'display:none' : ''">
						<image 
							class="selector-image" 
							:src="selectedItems.includes(index) ? 
								  '/static/选中.png' : 
								  '/static/未选中.png'"
						></image>
					</view>
					<image class="item-image" :src="item.image"></image>
					<!-- 商品详情 -->
					<view class="item-details">
					  <text class="item-name">{{ item.name }}</text>
					  <view class="item-price">
						<text class="label">价格:</text>
						<text class="price">¥{{ item.price }}</text>
					  </view>
					  <view class="item-quantity">
						<text class="label">数量(100g):</text>
						<text class="quantity">{{ item.quantity }}</text>
					  </view>
					</view>
					<view class="quantity-change">
						<image 
							src="/static/减号.png" 
							class="quantity-image-left"
							@click.stop="changeQuantity(index, -1)"
						></image>
						<text>{{item.quantity}}</text>
						<image 
							src="/static/加号.png" 
							class="quantity-image-right"
							@click.stop="changeQuantity(index, 1)"
						></image>
					</view>
				</view>
			</scroll-view>
			<!-- 底部结算栏（仅当有商品时显示） -->
			<view class="cart-footer" v-if="cartItems.length > 0">
				<view class="total-amount">
					<text class="label">总价(元):</text>
					<text class="amount">¥{{ totalPrice }}</text>
				</view>
				<button class="buy-button" @click="checkout">立即购买</button>
			</view>
		<my-tab-bar></my-tab-bar>
	</view>
</template>

<script>
	export default {
		data(){
			return{
				cartItems: [],		// 购物车商品列表
				selectedItems:[]	// 被选中的商品索引数组
			}
		},
		computed:{
			// 计算购物车总价
			totalPrice(){
				if (this.selectedItems.length > 0) {
					return this.selectedItems.reduce((total, index) => {
						const item = this.cartItems[index];
						return total + (item.price * item.quantity);
					}, 0).toFixed(2);
				} 
				else{
					return this.cartItems.reduce((total,item)=>{
						return total + (item.price * item.quantity);
					},0).toFixed(2);
				}
			},
			// 全选状态判断
			isAllSelected() {
				if (this.cartItems.length === 0) return false;
				return this.selectedItems.length === this.cartItems.length;
			}
		},
		onShow() {
			// 页面显示时加载购物车数据
			this.loadCartData();
			    
			// 监听购物车更新事件
			uni.$on('cart-updated', this.loadCartData);
		},
		onHide(){
			//页面隐藏时移除监听
			uni.$off('cart-updated',this.loadCartData);
		},
		methods:{
			//从本地存储加载购物车数据
			loadCartData(){
				this.cartItems = uni.getStorageSync('cart') || [];
			},
			//结算购物车
			checkout(){
				if(this.cartItems.length === 0){
					uni.showToast({
						title:'购物车为空',
						icon:'none'
					});
					return
				}
				// 如果有选中项目，则只结算选中项目
				if (this.selectedItems.length > 0) {
					const selectedCartItems = this.selectedItems.map(index => this.cartItems[index]);
					
					uni.navigateTo({
						url: '/pages/checkout/checkout?items=' + encodeURIComponent(JSON.stringify(selectedCartItems))
					});
				} else {
					// 否则结算所有商品
					uni.navigateTo({
						url: '/pages/checkout/checkout'
					});
				}
			},
			// 跳转到商品详细页
			goToContentPage(item) {
				uni.navigateTo({
					url: `/pages/content/content?keyword=${item.name}&source=cart`
				});
			},
			
			// 添加跳转到首页的方法
			goToIndex() {
				uni.navigateTo({
					url: '/pages/index/index'
				});
			},
			
			// 切换单个商品选中状态
			toggleSelectItem(index) {
				if (this.selectedItems.includes(index)) {
					this.selectedItems = this.selectedItems.filter(i => i !== index);
				} else {
					this.selectedItems.push(index);
				}
			},
			
			// 判断全选状态
			toggleSelectAll() {
				if (this.isAllSelected) {
					// 如果已经是全选状态，则取消全选
					this.selectedItems = [];
				} else {
					// 否则选中所有商品
					this.selectedItems = Array.from({ length: this.cartItems.length }, (_, i) => i);
				}
			},
			
			// 修改商品数量
			changeQuantity(index, step) {
				const newQuantity = this.cartItems[index].quantity + step;
				// 确保数量不小于1
				if(newQuantity >= 1) {
					this.cartItems[index].quantity = newQuantity;
					// 更新本地存储
					uni.setStorageSync('cart', this.cartItems);
				} else {
					uni.showToast({
						title: '商品数量不能少于1',
						icon: 'none'
					});
				}
			},
			
			// 倒序删除选中商品
			deleteSelectedItems() {
				if (this.selectedItems.length === 0) {
					uni.showToast({
						title: '请选择要删除的商品',
						icon: 'none'
					});
					return;
				}
				
				// 复制选中索引并排序（从大到小）
				const sortedIndexes = [...this.selectedItems].sort((a, b) => b - a);

				sortedIndexes.forEach(index => {
					this.cartItems.splice(index, 1);
				});

				// 更新存储
				uni.setStorageSync('cart', this.cartItems);

				// 清空选中项
				this.selectedItems = [];

				uni.showToast({
					title: `已删除${sortedIndexes.length}件商品`,
					icon: 'success'
				});
			},
			
			checkout(){
				// 显示成功弹窗
				uni.showToast({
					title: '购物成功！', // 提示内容
					icon: 'success',    // 成功图标
					duration: 2000,      // 2秒后自动关闭
				});
			}
		}
	}
</script>

<style scoped>
	.fixed-layout {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		overflow: hidden; /* 禁止滚动 */
	}
	
	.empty-cart {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: calc(100vh - 250rpx);
		padding: 20rpx;
		text-align: center;
		overflow: hidden;
	}
	
	.empty-image {
		width: 300rpx;
		height: 300rpx;
		margin-bottom: 30rpx;
		opacity: 0.6;
	}
	
	.empty-text {
		font-size: 36rpx;
		font-weight: bold;
		color: #6b6b6b;
		margin-bottom: 15rpx;
	}
	
	.empty-tip {
		font-size: 28rpx;
		color: #949494;
		margin-bottom: 40rpx;
	}
	
	.go-shopping-button {
		background-color: #e54d42;
		color: white;
		border-radius: 40rpx;
		padding: 0 60rpx;
		height: 80rpx;
		line-height: 80rpx;
		font-size: 32rpx;
	}
	
	.cart-header {
	  height: 60rpx;
	  padding: 20rpx;
	  background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
	  display: flex;
	  justify-content: space-between;
	  align-items: center;
	  border-bottom: 1px solid #eee;
	}
	
	.cart-title {
	  font-size: 36rpx;
	  font-weight: bold;
	}
	
	.cart-content{
		display: flex;
		flex-direction:row;
		justify-content: center;
		align-items: center;
	}
	
	.selected-all{
		height: 80rpx;
		width: 100rpx;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		margin-right: 20rpx;
	}
	
	.selector-all-image{
		width: 40rpx;
		height: 40rpx;
	}
	
	.cart-count {
	  font-size: 28rpx;
	  color: #666;
	}
	
	.cart-container {
	  height: calc(100vh - 300rpx);
	  background-color: #f1f1f1;
	}
	
	.cart-item {
	  display: flex;
	  background-color: #fff;
	  border-radius: 10rpx;
	  margin-bottom: 20rpx;
	  box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.1);
	}
	
	.selector-container {
		width: 60rpx;
		height: 166rpx;
		margin: 0 10rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}
		
	.selector-image {
	    width: 50rpx;
	    height: 50rpx;
	}
	
	.selector {
		width: 36rpx;
		height: 36rpx;
		border-radius: 50%;
		border: 2rpx solid #ccc;
		background-color: transparent;
		transition: all 0.2s ease;
	}
	
	.selector.selected {
		background-color: #e54d42;
		border-color: #e54d42;
		position: relative;
	}
	
	.selector.selected::after {
		content: "";
		position: absolute;
		left: 50%;
		top: 50%;
		width: 20rpx;
		height: 10rpx;
		border: 4rpx solid white;
		border-top: none;
		border-right: none;
		transform: translate(-50%, -60%) rotate(-45deg);
	}
	
	.item-image {
	  width: 160rpx;
	  height: 160rpx;
	  border-radius: 8rpx;
	  background-color: #f8f8f8;
	}
	
	.item-details {
	  flex: 1;
	  margin-left: 20rpx;
	  display: flex;
	  flex-direction: column;
	  justify-content: space-around;
	}
	
	.item-name {
	  font-size: 34rpx;
	  font-weight: bold;
	  margin-bottom: 10rpx;
	  color: #333;
	}
	
	.item-price, .item-quantity {
	  display: flex;
	  align-items: center;
	}
	
	.label {
	  font-size: 28rpx;
	  color: #797979;
	  margin-right: 10rpx;
	}
	
	.price, .quantity {
	  font-size: 32rpx;
	  font-weight: 400;
	  color: #797979;
	}
	
	.quantity-change{
		display: flex;
		flex-direction: row;
		height: 166rpx;
		justify-content: center;
		align-items: center;
		padding-right: 30rpx;
	}
	
	.quantity-image-left{
		width: 45rpx;
		height: 45rpx;
		margin: 0 10rpx;
	}
	
	.quantity-image-right{
		width: 50rpx;
		height: 50rpx;
		margin: 0 10rpx;
	}
	
	.cart-footer {
	  position: fixed;
	  bottom: 160rpx;
	  left: 0;
	  right: 0;
	  height: 100rpx;
	  background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
	  display: flex;
	  justify-content: space-between;
	  align-items: center;
	  padding: 0 30rpx;
	  border-top: 1px solid #eee;
	  z-index: 100;
	}
	
	.total-amount {
	  display: flex;
	  align-items: center;
	  flex-grow: 1;
	}
	
	.amount {
	  font-size: 36rpx;
	  font-weight: bold;
	  color: #e54d42;
	}
	
	.buy-button {
	  background-color: #e54d42;
	  color: white;
	  border-radius: 40rpx;
	  padding: 0 40rpx;
	  height: 70rpx;
	  line-height: 70rpx;
	  font-size: 32rpx;
	  margin-left: auto;
	  min-width: 200rpx;
	}
	
	.delete-box {
	    width: 80rpx;
	    height: 80rpx;
	    margin-right: 20rpx;
	    display: flex;
	    align-items: center;
		justify-content: center;
	    border-radius: 10rpx;
	    background-color: #f5f5f5;
	    box-shadow: 0 2rpx 6rpx rgba(0,0,0,0.1);
	}
	
	.trash-icon {
		width: 40rpx;
		height: 40rpx;
	}
</style>