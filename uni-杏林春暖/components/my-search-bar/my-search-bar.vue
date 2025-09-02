<template>
	<view class="search-container">
		<view class="search-bar">
			<image src="/static/搜索栏-01.png" mode="aspectFill"></image>
			<view class="placeholder-container">
				<!-- 动画时长300ms，动画滑动方向，自定义样式 -->
				<uni-transition 
					:show="showPlaceholder"
					:duration="300"
					:mode-class="animationDirection"
					:styles="transitionStyles">
						<input 
							type="text"
							ref="searchInput"
							@focus="handleFocus" 
							@blur="handleBlur" 
							@input="handleInput"
							:disabled="isAnimating"
							:placeholder="currentPlaceholder"
							v-model="inputValue"
							/>
				</uni-transition>
			</view>
			<button @click="performSearch">搜索</button>
		</view>
		<view class="suggestion-list" v-if="matchedItems.length>0">
			<scroll-view scroll-y="true" class="list-container">
				<view
					v-for="(item,index) in matchedItems"
					:key="index"
					class="suggestion-item"
					@tap="selectItem(item)"
				>
					{{item}}
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import { nameData } from '@/data/herbData';
	
	export default{
		data(){
			return{
				placeholderIndex:0,		// 当前提示词索引
				placeholders:nameData,  // 绑定数据集
				timer:null,				// 轮播定时器
				showPlaceholder: true,  // 控制动画元素显示
				animationDirection:'slide-top',	// 动画方向（初始向上滑出）
				isAnimating: false,     // 动画状态锁（防止动画冲突）
				transitionStyles: {		// 动画容器样式
					display: 'flex',
					alignItems: 'center',
					height: '100%',
					width: '100%',
				},
				inputValue:'',
				matchedItems:[],	// 匹配的药材列表
				isFocused:false,	// 输入框聚焦状态
				isSelecting:false,	//选择状态锁（防止失焦冲突）
				inputTimer:null		// 输入防抖定时器
			};
		},
		computed: {
			// 动态计算当前提示词
			currentPlaceholder() {
				return nameData[this.placeholderIndex];
			}
		},
		methods:{
			// 输入框聚焦事件处理
			handleFocus(){
				this.pauseRotation();	// 暂停占位符轮播
				this.isFocused = true;	// 标记聚焦状态
				// 已有输入内容时显示建议列表
				if(this.inputValue){
					this.filterSuggestions();
				}
			},
			
			// 输入框失焦事件处理
			handleBlur(){
				// 延迟300ms处理，避免与选择状态锁冲突
				setTimeout(()=>{
					if (this.$refs.searchInput && !this.isSelecting) {
					    this.$refs.searchInput.blur();
					}
					this.resumeRotation();	// 恢复轮播
				},300)
			},
			
			// 输入事件处理
			handleInput(event){
				this.inputValue = event.detail.value; // 更新输入值
				// 添加防抖处理（300ms延迟）
				clearTimeout(this.inputTimer);
				this.inputTimer = setTimeout(() => {
					this.filterSuggestions();
				}, 300);
			},
			
			// 过滤建议项
			filterSuggestions(){
				if(!this.inputValue){
					this.matchedItems = [];
					return
				}
				const keyword = this.inputValue.toLowerCase();
				// 过滤包含关键词的药材名称
				this.matchedItems = nameData.filter(item => 
				    item.toLowerCase().includes(keyword) // 包含即匹配
				);
			},
			
			// 选择建议项
			selectItem(item){
				this.isSelecting = true;	// 标记选择状态
				this.inputValue = item;
				this.matchedItems = [];
				// 强制重新渲染输入框
				this.inputKey = Date.now();	// 每次更新唯一标识
				this.performSearch();	// 执行搜索
			},
			
			// 执行搜索
			performSearch(){
				if(this.inputValue){
					console.log("执行搜索：",this.inputValue);
					uni.navigateTo({
						url: `/pages/content/content?keyword=${encodeURIComponent(this.inputValue)}`,
					});
					this.inputValue='';
				}
			},
			
			// 启动占位符轮播
			startRotation() {
			    this.timer = setInterval(() => {
				if (this.isAnimating) return; 	// 动画中跳过
				this.isAnimating = true;		// 锁定状态
				
				// 触发向上滑出动画
				this.animationDirection = 'slide-top';
				this.showPlaceholder = false;	// 隐藏元素（触发离开动画）
				
				setTimeout(() => {
					// 循环更新索引
					this.placeholderIndex = (this.placeholderIndex + 1) % this.placeholders.length;
					this.showPlaceholder = true;	// 显示新词（触发进入动画）
				  
					setTimeout(() => {
						// 动画结束后解锁状态
						this.isAnimating = false;
				  }, 300);	// 等待进入动画完成
				}, 300);	// 等待离开动画完成
			  }, 3000);	// 每3秒轮播一次
			},
			pauseRotation(){
				clearInterval(this.timer)//	清除定时器（暂停轮播）
			},
			resumeRotation() {
				this.pauseRotation();
				this.startRotation(); // 重启轮播
			}
		},
		mounted(){
			this.startRotation();	// 组件挂载时启动轮播
		},
		beforeUnmount(){
			this.pauseRotation();	// 组件销毁前清理定时器（防内存泄漏）
		}
	}
</script>

<style scoped>
	.search-bar{
		display: flex;
		justify-content: center;
		align-items: center;
		height: 90rpx;
		background: linear-gradient(135deg, #f8f4e5, #e8f5e9);
		position: relative;
		z-index: 60;
		padding: 0 10rpx 0 10rpx;
	}
	
	.search-bar image{
		width: 60rpx;
		height: 40rpx;
		z-index: 20;
	}
	  
	.search-bar input{
		height: 100%;
		width: 100%;
		font-size: 32rpx;
		color: #333;
		background: transparent;
		z-index: 15;
	}
	
	.search-bar button {
		min-width: 120rpx;
		height: 60rpx;
		padding: 0 20rpx;
		background-color: #4CAF50;
		color: #fff;
		border-radius: 30rpx;
		font-size: 24rpx;
		z-index: 25;
	}
	
	.search-bar::after {
	  content: '';
	  position: absolute;
	  top: 0;
	  right: 0;
	  bottom: 0;
	  left: 0;
	  pointer-events: none;
	  z-index: 5;
	}
	
	/* 修复对齐问题 */
	.uni-transition__content {
	  display: flex;
	  align-items: center;
	}
	
	.placeholder-container {
		flex: 1;
		height: 80rpx;
		overflow: hidden;
		position: relative;
		margin: 20rpx 20rpx;
	}  
	
	.suggestion-list {
	    position: absolute; 
	    top: 85%; /* 从搜索栏底部开始 */
	    left: 0;
	    right: 0;
	    z-index: 140; /* 确保在搜索栏下方 */
	    background-color: #f8f4e5;
		border-radius: 20rpx;
	}
	
	.list-container {
	    max-height: 400rpx;
	}
	  
	.suggestion-item {
	    padding: 20rpx 30rpx;
	    font-size: 28rpx;
	    border-bottom: 1rpx solid #eee;
	}
	  
	/* 建议项点击反馈 */
	.suggestion-item:active {
	    background-color: #f5f5f5;	
	}
	  
	.search-container {
	    position: relative;
		margin: 0 auto; 
		width: 100%;
	}
</style>