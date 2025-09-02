<template>
	<view class="container">
		<!-- 骨架屏 -->
		<view v-if="showContentSkeleton" class="skeleton-container">
			<!-- 模拟瀑布流布局 -->
			<view class="skeleton-column" v-for="i in 2" :key="i">
				<view class="skeleton-item" v-for="j in 5" :key="j">
					<!-- 图片占位 -->
					<view class="skeleton-image"></view>
					<!-- 文字占位 -->
					<view class="skeleton-text-line name"></view>
					<view class="skeleton-text-line property"></view>
					<view class="skeleton-text-line effect"></view>
				</view>
			</view>
			<!-- 加载进度条 -->
			<view class="skeleton-loading-indicator">
				<text class="loading-text">加载中 {{ loadingProgress }}%</text>
				<view class="loading-bar-container">
					<view 
						class="loading-bar-progress" 
						:style="{ width: `${loadingProgress}%` }">
					</view>
				</view>
			</view>
		</view>
		
		<view class="visible-container">
			<!-- 瀑布流列容器：v-for生成指定列数 -->
			<view class="column" v-for="(col, colIndex) in columnData" :key="colIndex">
				<view
					v-for="item in col"
					:key="item.id"
					class="medicine-box"
					@click="goToContent(item.name)"
				>
					<!-- 图片 -->
					<image
						:src="getMedicineImage(item.name)"
						mode="aspectFill"
						class="medicine-image"
						lazy-load="true"
					></image>
					<!-- 商品信息 -->
					<view class="medicine-info">
						<view class="text-container"> 
							<text class="medicine-name">{{ item.name }}</text>
						</view>
						<view class="text-container"> 
							<text class="medicine-property">性味：{{ item.flavor }}</text>
						</view>
						<view class="text-container"> 
							<text class="medicine-effect">功效：{{ item.effect }}</text>
						</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 加载指示器 -->
		<view v-if="isLoading" class="loading-indicator">加载中 {{ getLoadingProgress() }}%</view>
		<view v-if="!hasMore" class="no-more-indicator">没有更多了</view>
	</view>
</template>

<script>
	import { nameData } from '@/data/herbData.js';
	const apiCache = {};	// API响应缓存池（减少重复请求）
  
	export default {
		data() {
			return {
				columnCount: 2,          	// 列数
				columnData: [[], []], 	 	// 确保每列都有数组容器
				allMedicines: [],        	// 所有药品数据
				isLoading: false,			// 加载状态标志
				hasMore: true,           	// 是否有更多数据
				currentPage: 1,          	// 当前页码
				pageSize: 20,            	// 每页数据量
				showContentSkeleton: true, 	// 骨架屏显示状态
				loadingProgress: 0,         // 加载进度百分比
				loadedIds: new Set()		// 已加载ID集合（用于去重）
			};
		},
	
		mounted() {
			// 初始加载数据
			this.loadMedicines();
			console.log("药材总数:", nameData.length);
			
			// 添加滚动监听
			uni.$on('scrollToBottom', this.handleScrollToBottom);
		},
		
		beforeDestroy() {
			// 移除滚动监听
			uni.$off('scrollToBottom', this.handleScrollToBottom);
		},
    
		methods: {
			// 处理滚动到底部事件
			handleScrollToBottom() {
				if (!this.isLoading && this.hasMore) {
					this.loadMedicines();
				}
			},
			
			goToContent(name){
				uni.navigateTo({
					url: `/pages/content/content?keyword=${name}&source=index`
				});
			},
		
			// 加载药品数据
			async loadMedicines() {
				if (this.isLoading || !this.hasMore) return;
				    
				    this.isLoading = true;
				    this.showContentSkeleton = true;
				    
				    try {
				        // 直接使用所有nameData，不再分页
				        const newData = await Promise.all(
				            nameData.map(async (name) => {
				                // 检查是否已加载过该数据
				                if (this.loadedIds.has(name)) {
				                    return null; // 已加载则跳过
				                }
				                
				                const apiData = await this.cachedRequest(name);
				                this.loadedIds.add(name); // 标记为已加载
				                
				                return {
				                    id: name, // 直接使用name作为ID
				                    name,
				                    image: this.getMedicineImage(name),
				                    flavor: apiData?.flavor || apiData?.property || '暂无数据',
				                    effect: apiData?.effect || apiData?.function || '暂无数据'
				                };
				            })
				        );
				        
				        // 过滤掉null值和无效数据
				        const validNewData = newData.filter(item => item !== null && item.name);
				        
				        // 合并数据
				        this.allMedicines = [...this.allMedicines, ...validNewData];
				        
				        // 更新瀑布流布局
				        this.updateVisibleItems();
				        
				        // 更新加载进度
				        this.loadingProgress = Math.min(
				            Math.floor((this.loadedIds.size / nameData.length) * 100),
				            100
				        );
				        
				        console.log("加载数据量:", validNewData.length);
				        console.log("当前总数据:", this.allMedicines.length);
				        
				        // 检查是否还有更多数据
				        if (this.loadedIds.size >= nameData.length) {
				            this.hasMore = false;
				            console.log('所有药材数据加载完成');
				        }
				} catch (error) {
					console.error('加载失败', error);
					// 添加错误处理UI反馈
					uni.showToast({
						title: '数据加载失败',
						icon: 'none',
						duration: 2000
					});
				} finally {
					this.isLoading = false;
					// 关闭骨架屏
					setTimeout(() => {
						this.showContentSkeleton = false;
					}, 500);
				}	
			},
			
			// 更新可见项目（瀑布流分列）
			updateVisibleItems() {
				// 重置列数据
				    this.columnData = Array.from({ length: this.columnCount }, () => []);
				    const columnHeights = Array(this.columnCount).fill(0);
				    
				    // 按列高度动态分配项目
				    this.allMedicines.forEach(item => {
				        // 找出当前高度最小的列
				        const minHeightIndex = columnHeights.indexOf(Math.min(...columnHeights));
				        
				        // 将项目添加到该列
				        this.columnData[minHeightIndex].push(item);
				        
				        // 估算并更新该列的高度（假设每个项目高度大约为420rpx）
				        columnHeights[minHeightIndex] += 420;
				    });
				    
				    // 强制更新视图
				    this.$forceUpdate();
			},
		
			// 获取药材图片
			getMedicineImage(name) {
				const imagePath = `/static/herb/${name}.jpg`;
				return imagePath;
			},
		
			// 加载进度指示器
			getLoadingProgress() {
				return Math.min(
					Math.floor((this.allMedicines.length / nameData.length) * 100), 
					100
				);
			},
	  
			// 添加缓存请求函数
			async cachedRequest(name) {
				if (apiCache[name]) return apiCache[name];
				try {
					// 完整响应对象
					const response = await uni.request({
						url: `http://api.zhyunxi.com/api.php?api=87&key=d51b5be2f98dd9bdf06e0bb47dea87fc&name=${encodeURIComponent(name)}`,
						timeout: 10000 ,// 延长至10秒
						dataType: 'json', // 强制JSON解析
					});
				  
					// 兼容两种响应结构
					const res = response[1] ? response[1] : response; // 关键修复点
				  
					// 增强验证逻辑
					if (!res || res.statusCode !== 200 || !res.data || res.data.code !== 0) {
						console.error(`[API异常] ${name}`, { 
						  status: res.statusCode, 
						  data: res.data || '无响应数据'
						});
						return null;
					}
				  
					// 提取有效数据
					const validData = Array.isArray(res.data.data) && res.data.data.length > 0 
						? res.data.data[0] 
						: null;
				  
					// 新增字段有效性验证
					if (validData) {
						// 关键字段缺失检测
						const isValid = validData.flavor || validData.effect || validData.property || validData.function;
						if (!isValid) {
							console.warn(`[数据字段缺失] ${name}`, validData);
							return { 
							  flavor: '数据异常', 
							  effect: '数据异常',
							  _rawData: validData // 保留原始数据用于调试
							};
						}
					}
					apiCache[name] = validData;
					return validData;
				} catch (e) {
				  console.error(`[网络异常] ${name}`, e.errMsg || e);
				  return null;
				}
			}
		}
	};
</script>

<style>
	/* 样式部分保持不变，与您原来的代码一致 */
	.container {
		display: flex;
		flex-direction: column;
		flex-grow: 1; /* 填充空间 */
		width: 100%;
		min-height: 100vh; 
		position: relative;
		padding-bottom: 75rpx;
	}
  
	.skeleton-container {
		padding: 20rpx;
		display: flex;
		gap: 20rpx;
	}
	
	.skeleton-column {
		flex: 1;
	}
	
	.skeleton-item {
		margin-bottom: 30rpx;
		background: #fff;
		border-radius: 16rpx;
		overflow: hidden;
		box-shadow: 0 6rpx 18rpx rgba(0,0,0,0.08);
		padding: 15rpx;
	}
	
	.skeleton-image {
		height: 240rpx;
		background: linear-gradient(to right, #f5f5f5, #e0e0e0, #f5f5f5);
		background-size: 200% 100%;
		animation: loadingAnimation 1.5s infinite linear;
		border-radius: 8rpx;
		margin-bottom: 20rpx;
	}
	
	.skeleton-text-line {
		height: 30rpx;
		background: linear-gradient(to right, #f5f5f5, #e0e0e0, #f5f5f5);
		background-size: 200% 100%;
		animation: loadingAnimation 1.5s infinite linear;
		border-radius: 4rpx;
		margin-bottom: 10rpx;
	}
	
	.skeleton-text-line.name {
		width: 70%;
		height: 40rpx;
		margin-bottom: 8rpx;
	}
	
	.skeleton-text-line.property {
		width: 50%;
	}
	
	.skeleton-text-line.effect {
		width: 80%;
	}
	
	/* 加载指示器 */
	.skeleton-loading-indicator {
		position: absolute;
		bottom: 20rpx;
		left: 50%;
		transform: translateX(-50%);
		width: 80%;
		background: rgba(255, 255, 255, 0.9);
		padding: 15rpx 20rpx;
		border-radius: 12rpx;
		box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.1);
		z-index: 100;
	}
	
	.loading-text {
		font-size: 28rpx;
		color: #666;
		display: block;
		text-align: center;
		margin-bottom: 8rpx;
	}
	
	.loading-bar-container {
		height: 10rpx;
		background: #f0f0f0;
		border-radius: 5rpx;
		overflow: hidden;
	}
	
	.loading-bar-progress {
		height: 100%;
		background: linear-gradient(90deg, #4cd964, #34e89e);
		transition: width 0.3s ease;
		border-radius: 5rpx;
	}
	
	/* 骨架屏动画 */
	@keyframes loadingAnimation {
		0% { background-position: 100% 50%; }
		100% { background-position: 0 50%; }
	}
  
	.visible-container {
		position: relative;
		display: flex;
		width: 100%;
		transform: translateZ(0); 	/* 开启GPU加速 */
		will-change: transform;     /* 提示浏览器提前优化 */
		margin-bottom: 0; /* 移除底部外边距 */
	}
  
	.text-container {
		display: block; /* 独占一行 */
		width: 100%;    /* 撑满父容器宽度 */
		overflow: hidden; 
		text-overflow: ellipsis;
		white-space: nowrap; /* 禁止换行 */
	}
  
	.column {
		flex: 0 0 50%; /* 固定宽度50%，不再自适应 */
		min-width: 0;
		padding: 0 10rpx;
		box-sizing: border-box; /* 包含padding在宽度内 */
	}
	
	.text-container {
	    display: block;
	    width: 100%;
	    text-overflow: ellipsis;
		overflow: hidden; /* 保证单行显示 */
	    white-space: nowrap;
	}
  
	.medicine-box {
		height: 420rpx;
		margin-bottom: 20rpx;
		background: #fff;
		border-radius: 16rpx;
		overflow: hidden;
		box-shadow: 0 6rpx 18rpx rgba(0,0,0,0.08);
		will-change: transform;
		contain: strict; /* 隔离渲染影响 */
	}
  
	.medicine-image {
		width: 100%;
		height: 240rpx;
		background: linear-gradient(to right, #f6f7f8, #e9ebee, #f6f7f8);
		background-size: 200% 100%;
	}
  
	.medicine-info {
		padding: 15rpx;
	}
  
	.medicine-name {
		display: block;
		font-size: 32rpx;
		font-weight: bold;
		margin-bottom: 8rpx;
	}
  
	.medicine-property {
		font-size: 26rpx;
		color: #666;
	}
  
	.loading-indicator,
	.no-more-indicator {
		position: relative; /* 改为相对定位 */
		bottom: auto;
		left: auto;
		right: auto;
		text-align: center;
		padding: 20rpx;
		color: #999;
		background: rgba(228, 228, 228, 0.9);
		z-index: 60;
		margin-bottom: 20rpx;
		width: 100%;
		box-sizing: border-box;
		margin-bottom: 100rpx;
	}
	
	.medicine-effect {
		display: block;
		font-size: 26rpx;
		color: #000000;
		margin-top: 5rpx;
	}
</style>