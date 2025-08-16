<template>
	<view class="container">
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
				<!-- 动态绑定ID用于高度测量 -->
				<view
				v-for="item in col"
				:key="item.id"
				class="medicine-box"
				@click="goToContent(item.name)"
				:id="'box-'+item.id"
				>
					<!-- 5.图片加载后更新高度 -->
					<image
					  :src="getMedicineImage(item.name)"
					  mode="aspectFill"
					  class="medicine-image"
					  lazy-load="true"
					  @load="e => updateItemHeight(e, item.id)"
					></image>
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
      
			<!-- 加载指示器 -->
			<view v-if="isLoading" class="loading-indicator">加载中 {{ getLoadingProgress() }}%</view>
			<view v-if="!hasMore" class="no-more-indicator">没有更多了</view>
		</view>
	</view>
</template>

<script>
  import { nameData } from '@/data/herbData.js';
  const apiCache = {};	// API响应缓存池（减少重复请求）
  
  export default {
    data() {
      return {
        columnCount: 2,          // 列数
        columnData: [[], []], 	 // 确保每列都有数组容器
        allMedicines: [],        // 所有药品数据
        scrollViewHeight: 600,   // 滚动容器高度
        scrollTop: 0,            // 滚动位置
        itemHeight: 360,         // 每个项目预估高度(px)
        visibleCount: 0,         // 可视区域内显示的项目数
        startIndex: 0,           // 当前显示的起始索引
        offsetTop: 0,            // 内容偏移量
        isLoading: false,
		loadThreshold:5,
		endIndex: 0,			 // 改为普通响应式属性
		loadQueue: 0, 			 // 防止递归过深
		maxLoadQueue: 5,		 // 最大递归深度
        hasMore: true,           // 是否有更多数据
        currentPage: 1,          // 当前页码
        pageSize: 40,            // 每页数据量
		heightTimers: {},        // 存储高度计算定时器
		showContentSkeleton: true, // 骨架屏显示状态
		loadingProgress: 0          // 加载进度百分比
      };
    },
	
	mounted() {
	// 初始加载数据
	this.loadMedicines();	// 组件挂载立即加载数据
    this.initScrollView();	// 初始化滚动容器尺寸
	console.log("药材总数:", nameData.length);  
    },
	
    computed: {
		// 计算列表总高度
		totalHeight() {
			return this.allMedicines.length * this.itemHeight;
		},

		// 动态计算顶部padding
		dynamicPaddingTop() {
			return Math.max(0, this.headerHeight - this.scrollOffset);
		}
    },
    
    methods: {
		goToContent(name){
			uni.navigateTo({
				url: `/pages/content/content?keyword=${name}&source=index`
			});
		},
		// 初始化滚动容器
		initScrollView() {
			// 使用选择器获取实际高度
			this.$nextTick(() => {
				const query = uni.createSelectorQuery().in(this);
				query.select('.virtual-scroll-container').boundingClientRect(rect => {
					if (rect) {
						this.scrollViewHeight = rect.height;
						// 计算可视项数量：容器高度/单项高度 + 缓冲区
						this.visibleCount = Math.max(1, Math.ceil(this.scrollViewHeight / this.itemHeight) + 5);
					}
				}).exec();
			});
		},
	  
		// 计算可视区域                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                
		calculateVisibleRange(scrollTop) {
			if (Date.now() - this.lastScrollTime < 100) return; // 100ms节流
			const itemsPerRow = this.columnCount; // 每行项目数
			
			// 计算结束索引（基于可视行数+缓冲）
			const visibleRows = Math.ceil(this.scrollViewHeight / this.itemHeight);
			const visibleItemsCount = visibleRows * this.columnCount; 
			this.startIndex = Math.max(0, 
				Math.floor(scrollTop / this.itemHeight) * itemsPerRow - 10 // 缓冲10个项目
			);
			this.endIndex = this.allMedicines.length; // 直接渲染所有已加载数据
			
			// 确保初始化 lastStartIndex
			if (typeof this.lastStartIndex === 'undefined') {
				this.lastStartIndex = this.startIndex;
			}
			
			// 仅当索引变化超过30个项时更新
			if (Math.abs(this.startIndex - this.lastStartIndex) > 30) {
			  this.updateVisibleItems();
			  this.lastStartIndex = this.startIndex;
			}
			
			// 检查是否需要加载更多
			if (this.shouldLoadMore()) {
			  this.loadMedicines();
			}
			
			// 动态偏移量
			this.offsetTop = this.startIndex * this.itemHeight;
		},
      
		// 更新可见项目
		updateVisibleItems() {
			// 按列清空旧数据
			this.columnData.forEach(col => col.length = 0);
			const safeEndIndex = Math.min(this.endIndex, this.allMedicines.length);
			const visibleItems = this.allMedicines.slice(this.startIndex, safeEndIndex);
			   
			// 确保每列都是有效数组
			for (let i = 0; i < this.columnCount; i++) {
				if (!Array.isArray(this.columnData[i])) {
				 this.columnData[i] = []; // 安全初始化
				}
				this.columnData[i].length = 0; // 清空列内容
			}
			   
			// 动态分列算法
			visibleItems.forEach((item, index) => {
				const columnIndex = index % this.columnCount;
				this.columnData[columnIndex].push({
					...item,
					id: `item-${this.startIndex + index}`
				});
			});
		},
		
		// 动态测量项目高度
		updateItemHeight(e, id) {
			clearTimeout(this.heightTimers[id]);
			this.heightTimers[id] = setTimeout(() => {
				uni.createSelectorQuery()
				    .in(this)
				    .select(`#box-${id}`)
				    .boundingClientRect(rect => {
					// 动态调整全局项高度基准
				    if (rect && rect.height > this.itemHeight) {
					    this.itemHeight = rect.height;
				    }}).exec();
			}, 200);
	    },
	  
        // 判断是否需要加载更多
        shouldLoadMore() {
			const remainingItems = this.allMedicines.length - this.endIndex;
			return(
				!this.isLoading &&
				this.hasMore &&
				remainingItems > 0
			);
        },
      
        // 加载药品数据
        async loadMedicines() {
			this.showContentSkeleton = true;
			this.loadingProgress = 0; // 重置进度
			if (this.isLoading || !this.hasMore) return;
			this.loadQueue++;
			this.isLoading = true;
			
			try {
			    const start = (this.currentPage - 1) * this.pageSize;
				// 先检查是否还有数据可加载
				if (start >= nameData.length) {
					this.hasMore = false;
					return;
				}
				
				// 模拟加载进度
				const totalPages = Math.ceil(nameData.length / this.pageSize);
				let currentProgress = 0;
				
				for (let page = 1; page <= totalPages; page++) {
					// 计算实际可加载的数据量
					const end = Math.min(start + this.pageSize, nameData.length);
					const names = nameData.slice(start, end);
				  
					// 并发请求API数据
					const newData = await Promise.all(names.map(async (name) => {
						const apiData = await this.cachedRequest(name);
					  
						return {
							id: `${this.currentPage}-${name}`,	// 唯一ID生成规则
							name,
							image: this.getMedicineImage(name),
							flavor: apiData?.flavor || apiData?.property || '暂无数据',
							effect: apiData?.effect || apiData?.function || '暂无数据'
						};
					}));
					
					// 合并数据（确保不重复）
					this.allMedicines = [...this.allMedicines, ...newData];
					this.currentPage++;
				  
					this.$nextTick(() => {
						this.columnData = Array.from({ length: this.columnCount }, () => []);
						// 初始加载后强制计算可视区域
						this.calculateVisibleRange(0); // 传入scrollTop=0
						this.updateVisibleItems(); // 立即更新可见项
					});
				  
					// 计算总页数
					const totalPages = Math.ceil(nameData.length / this.pageSize);
				  
					// 自动加载下一页（递归调用）
					if (this.allMedicines.length < nameData.length) {
						this.$nextTick(() => {
						this.loadMedicines();
					});
					} else {
						this.hasMore = false;
						console.log('所有药材数据加载完成');
					}
					console.log("加载数据量:", newData.length);
					console.log("当前总数据:", this.allMedicines.length);
					// 更新进度
					currentProgress = Math.floor((this.allMedicines.length / nameData.length) * 100);
					this.loadingProgress = Math.min(currentProgress, 100);
					await this.delay(100); // 添加延迟使进度可见
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
			  this.loadQueue--;
			  this.isLoading = false;
			  // 关闭骨架屏
			  setTimeout(() => {
				  this.showContentSkeleton = false;
			  }, 500);
			}	
        },
		
		// 简单延迟函数
		delay(ms) {
			return new Promise(resolve => setTimeout(resolve, ms));
		},
      
        // 获取药材图片
		getMedicineImage(name) {
			const imagePath = `/static/herb/${name}.jpg`;
			return imagePath;
		},
		
	    //加载性能指示器
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
	.container {
		display: flex;
		flex-direction: column;
		flex-grow: 1; /* 填充空间 */
		width: 100%;
		min-height: 100vh; 
		position: relative;
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
  
	.virtual-scroll-container {
		position: relative;
		height: 100%;
	}
  
	.visible-container {
		position: relative;
		display: flex;
		width: 100%;
		transform: translateZ(0); 	/* 开启GPU加速 */
		will-change: transform;     /* 提示浏览器提前优化 */
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
  
  .column-container {
      display: flex;
      width: 100%;
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
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		text-align: center;
		padding: 20rpx;
		color: #999;
		background: rgba(228, 228, 228, 0.9); /* 半透明背景 */
		z-index: 60;
	}
	
	.medicine-effect {
		display: block;
		font-size: 26rpx;
		color: #000000;
		margin-top: 5rpx;
	}
</style>
