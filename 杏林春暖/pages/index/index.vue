<template>
	<view class="page-container">  
		<my-nav-bar :title="'中医药大全'"></my-nav-bar>
		<!-- 监听滚动事件，使用.passive避免阻塞滚动。动态设置高度 -->
		<view class="page-container" @scroll.passive ="handleMainScroll" :style="{height: scrollViewHeight+'px'}">
			<view class="header-area">
				<my-search-bar ref="searchBar"></my-search-bar>
				<my-carousel-bar ref="carousel"></my-carousel-bar>
			</view>
				<my-container-bar></my-container-bar>
				<!-- buffer-size=30优化渲染性能 -->
				<my-content-bar ref="content" :buffer-size="30" class="content-box">
				</my-content-bar>
		</view>
		<my-topButton-bar  
			:showBackToTop="showBackToTop" 
			@back-to-top="handleBackToTop">
		</my-topButton-bar>
		<my-tab-bar></my-tab-bar>
	</view>
</template>

<script>
	export default {
		data() {
			return {
			   scrollTop: 0,           // 当前滚动位置
			   showBackToTop: false,   // 控制返回顶部按钮显示
			   scrollViewHeight: 0,    // 滚动容器高度
			   lastScrollTime: 0,	   // 滚动事件节流控制
			   navTotalHeight:44
			};
		},
		mounted() {
		    // 确保页面加载时在顶部
		    this.scrollTop = 0;
			
			const systemInfo = uni.getSystemInfoSync();
			this.scrollViewHeight = systemInfo.windowHeight - this.navTotalHeight;
			
			// 计算滚动容器高度
		    this.calculateScrollViewHeight();
		},
		methods: {
			// 计算滚动容器高度
			calculateScrollViewHeight() {
				// 获取设备信息
				const systemInfo = uni.getSystemInfoSync();
				// 设置为屏幕高度
				this.scrollViewHeight = systemInfo.windowHeight;
			},
			
			// 滚动事件处理
			handleMainScroll(e) {
				// 获取当前滚动位置
			    const scrollTop = e.detail.scrollTop;
				// 添加节流控制
				const now = Date.now();
				if (now - this.lastScrollTime < 100) return;
				this.lastScrollTime = now;
				   
				// 滚动超过150px显示返回按钮
				this.showBackToTop = scrollTop > 150;
					
				// 调试输出
				console.log(`滚动位置: ${scrollTop}, 显示状态: ${this.showBackToTop}`);
				// 更新当前滚动位置
				this.scrollTop = scrollTop;
				this.lastScrollPosition = scrollTop;
			
			},
			
			// 返回顶部操作
			handleBackToTop() {
				//重置滚动位置
				this.scrollTop = 0;
				// 隐藏按钮
				this.showBackToTop = false;
				// 双保险策略，同时使用数据绑定+API调用以解决真机兼容性问题
				uni.pageScrollTo({
				    scrollTop: 0,
				    duration: 360,
				    success: () => this.showBackToTop = false
				});
			}
		}
	};
</script>

<style scoped>
	.page-container {
		display: flex;
		flex-direction: column;
		background-color: #e4e4e4;
		height: calc(100vh - var(--nav-total-height)); /* 动态高度 */
	}
	
	.content-container {
		overflow: auto;
		flex:1;
		transform: translateZ(0); /* 开启GPU加速 */
	}
	
	.header-area {
		background-color: #e4e4e4;
	}
	
	.content-box{
		padding-top: 10rpx;
		background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
	}
</style>