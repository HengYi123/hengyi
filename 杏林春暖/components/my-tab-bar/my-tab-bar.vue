<template>
  <view class="custom-tab-bar">
    <view 
      v-for="(item, index) in tabList" 
      :key="index"
      class="tab-item"
      @click="switchTab(item,index)"
    >
      <image 
		class="tab-item-image"
		:src="currentTab === index ? item.selectedIcon : item.icon" />
      <text :class="{ active: currentTab === index }">{{ item.text }}</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      currentTab: 0,
      tabList: [
        {
          pagePath: "/pages/index/index",
          icon: "/static/button/首页01.png",
          selectedIcon: "/static/button/首页02.png",
          text: "首页"
        },
        {
          pagePath: "/pages/shoppingCart/shoppingCart",
          icon: "/static/button/购物车01.png",
          selectedIcon: "/static/button/购物车02.png",
          text: "购物车"
        },
        {
          pagePath: "/pages/my/my",
          icon: "/static/button/用户01.png",
          selectedIcon: "/static/button/用户02.png",
          text: "我的"
        }
      ]
    };
  },
  mounted(){
		// 监听全局页面显示事件
		uni.$on('tab-page-show',this.onTabPageShow);
		// 初始化时设置当前选中项
		this.updateActiveTab();
  },
  beforeDestroy() {
		// 组件销毁时移除监听
		uni.$off('tab-page-show', this.onTabPageShow);
    },
  methods: {
    switchTab(item,index) {
		// 立即更新UI响应
		this.currentTab = index;
		uni.reLaunch({ 
		    url: item.pagePath
		});
    },
	onTabPageShow(route) {
	    if (!route) return; // 确保路由存在
	        
		// 路径格式处理 (去掉前面的 "/")
		const path = route.startsWith('/') ? route : `/${route}`;
		
		if (!targetPath.startsWith('/pages')) {
		    targetPath = `/${targetPath}`;
		}
		
		// 匹配当前路径对应的tab索引
		const index = this.tabList.findIndex(item => 
		  item.pagePath === path || 
		  item.pagePath === `/${path}`
		);
		
		if (index !== -1) this.currentTab = index;
	},
	updateActiveTab() {
	    // 获取当前页面路径
	    const pages = getCurrentPages();
	    if (!pages.length) return;
	    const currentRoute = '/' + pages[pages.length - 1].route;
	      
	    // 匹配当前路径对应的tab索引
	    const activeIndex = this.tabList.findIndex(
			item => item.pagePath === currentRoute
	    );
	    if (activeIndex !== -1) this.currentTab = activeIndex;
	}
  }
};
</script>

<style scoped>
.custom-tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding-bottom: env(safe-area-inset-bottom); /* 适配全面屏手机 */
  height: 50px;
  background-color: #f8f8f8;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-top: 2px solid #e5e5e5;
}

.tab-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.tab-item-image {
  width: 24px;
  height: 24px;
  margin-bottom: 2px;
}

.active {
  color: #007aff;
}
</style>