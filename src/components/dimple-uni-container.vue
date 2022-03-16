<template>
  <view class="container" :style="{ height: computedHeight, background: background }">
    <!-- header -->
    <view ref="header" class="header" :style="{ top: headerTop + 'px' }">
      <!-- 自定义nav-bar -->
      <view v-if="customNavBar" class="nav-bar" :style="navBarStyle">
        <!-- 状态栏 -->
        <view v-if="statusBarHeight > 0" class="status-bar" :style="computedStatusBarStyle"></view>

        <!-- 标题栏 -->
        <view v-if="titleBarHeight > 0" class="title-bar" :style="computedTitleBarStyle">
          <!-- 标题栏左侧 -->

          <view :style="{ width: menuRect.width + 'px', height: titleBarHeight + 'px', marginLeft: menuRect.rightGap + 'px', marginRight: menuRect.rightGap + 'px' }">
            <slot name="titleLeft">
              <view class="back-icon-container" :style="{ height: titleBarHeight + 'px' }" @click="back">
                <image class="back-icon" :style="backIconStyle" :src="backIcon" mode="aspectFit" />
              </view>
            </slot>
          </view>

          <!-- 标题栏内容 -->
          <view class="title-bar-content" :style="{ height: titleBarHeight + 'px' }">
            <slot name="titleContent">
              <view class="title-bar-content-title" :style="{ lineHeight: titleBarHeight + 'px' }">
                {{ title }}
              </view>
            </slot>
          </view>

          <!-- 标题栏右侧 -->
          <view class="title-bar-right" :style="{ width: menuRect.width + 'px', height: titleBarHeight + 'px', marginRight: menuRect.rightGap + 'px', marginLeft: menuRect.rightGap + 'px' }">
            <slot name="titleRight"> </slot>
          </view>
        </view>
      </view>

      <!-- header slot-->
      <slot name="header"></slot>
    </view>

    <!-- header占位区域 -->
    <view class="header-placeholder" :style="{ height: headerPlaceholderHeight + 'px', background: background }"> </view>

    <!-- 内容区 -->
    <view class="content">
      <!-- 默认slot -->
      <slot></slot>
    </view>

    <!-- footer占位区域 -->
    <view v-if="footerPlaceholderHeight > 0" class="footer-placeholder" :style="{ height: footerPlaceholderHeight + 'px', background: background }"> </view>

    <!-- 安全底部占位区域 -->
    <view v-if="safeBottmHeight > 0" class="safe-bottom-placeholder" :style="{ height: safeBottmHeight + 'px' }"> </view>

    <!-- footer -->
    <view ref="footer" class="footer" :style="{ background: footerBackground, top: footerTop + 'px' }">
      <!-- footer slot -->
      <slot name="footer"></slot>

      <!-- footer安全底部占位区域  -->
      <view v-if="safeBottmHeight > 0" class="safe-bottom-placeholder" :style="{ background: footerBackground, height: safeBottmHeight + 'px' }"> </view>
    </view>

    <!-- 用于计算安全距离的元素，uni的统一api还存在未解决的bug，先使用幽灵元素计算安全距离 -->
    <view class="safe-bottom-ghost"> </view>
  </view>
</template>

<script>
let resizeObserver = null

export default {
  name: 'dimple-uni-container',
  props: {
    height: { type: String, default: '' }, // 高度，不设置默认自动计算
    background: { type: String, default: '#eeeeee' }, // 背景颜色
    footerBackground: { type: String, default: '#ffffff' }, // footer背景颜色, 安全占位区会同步这个背景色
    title: { type: String, default: '' }, // 自定义navbar的标题
    navBarStyle: { type: String, default: '' }, // 自定义navbar的样式
    statusBarStyle: { type: String, default: '' }, // 自定义navbar的状态栏样式
    titleBarStyle: { type: String, default: '' }, // 自定义navbar的标题栏样式
    customBack: { type: Boolean, default: false }, // 开启自定义监听返回事件的标识
    homeUrl: { type: String, default: '/pages/index/index' }, // 返回首页的地址
    backIcon: {
      // 返回的icon，支持网络地址和base64
      type: String,
      default:
        'data:image/svg+xml;base64,PHN2ZyBjbGFzcz0ibXlzdmciIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHdpZHRoPSI4Ljk5ODEwODUwNjU3NTEzNiIgaGVpZ2h0PSIxNyIgdmlld0JveD0iMCAwIDguOTk4MTA4NTA2NTc1MTM2IDE3IiBmaWxsPSJub25lIj48cGF0aCBpZD0i5Zu+5qCH6aKc6ImyIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiIHN0eWxlPSJmaWxsOiMwMDAwMDAiIHRyYW5zZm9ybT0idHJhbnNsYXRlKDIuMTY4NDA0MzQ0OTcxMDA5ZS0xOSAwKSAgcm90YXRlKDAgNC40OTkwNTQyNTMyODc1NzggOC41KSIgb3BhY2l0eT0iMC45IiBkPSJNMS42OCw4LjVMOSwxLjA2TDcuOTUsMEwwLjI4LDcuOEMtMC4xLDguMTkgLTAuMSw4LjgxIDAuMjgsOS4yTDcuOTUsMTdMOSwxNS45NEwxLjY4LDguNVogIj48L3BhdGg+PC9zdmc+',
    },
    backIconStyle: { type: String, default: 'height: 17px;width: 9px;' }, // 返回icon的样式
  },
  data() {
    return {
      systemInfo: {}, // 记录系统信息
      loading: true, // 加载标识
      containerHeight: {},
      headerPlaceholderHeight: 0, // header占位高度
      footerPlaceholderHeight: 0, // footer占位高度
      headerTop: 0,
      footerTop: 0,
      safeBottmHeight: 0, // 安全距离高度
      customNavBar: false, // 是否开启自定义navbar的标识
      statusBarHeight: 0, // 自定义navbar状态栏高度
      titleBarHeight: 0, // 自定义navbar标题栏高度
      menuRect: {}, // 小程序胶囊按钮的rect信息
    }
  },
  computed: {
    computedHeight() {
      if (this.height) return this.height
      return this.systemInfo.windowHeight + 'px'
    },
    computedNavBarStyle() {
      return `${this.navBarStyle}`
    },
    computedStatusBarStyle() {
      return `height: ${this.statusBarHeight}px;${this.statusBarStyle}`
    },
    computedTitleBarStyle() {
      return `height: ${this.titleBarHeight}px;${this.titleBarStyle}`
    },
    isH5() {
      // #ifdef H5
      return true
      // #endif
      return false
    },
    isMp() {
      // #ifdef MP-WEIXIN
      return true
      // #endif
      return false
    },
    isApp() {
      // #ifdef APP-PLUS
      return true
      // #endif
      return false
    },
    isIos() {
      return (this.systemInfo.platform + this.systemInfo.system).toLowerCase().indexOf('ios') > -1
    },
    isAndroid() {
      return (this.systemInfo.platform + this.systemInfo.system).toLowerCase().indexOf('android') > -1
    },
  },
  methods: {
    // 根据classname获取dom元素clientRect信息，并把方法promise化
    getDomRect(className) {
      const query = uni.createSelectorQuery().in(this)
      return new Promise((res) => {
        query
          .select(className)
          .boundingClientRect((data) => res(data))
          .exec()
      })
    },

    // 获取胶囊Rect信息，兼容写法
    getMenuButtonRect() {
      if (this.isH5 || this.isApp) return { height: 44, width: 90, topGap: 10, rightGap: 10 }
      const rect = uni.getMenuButtonBoundingClientRect()
      const { statusBarHeight, screenWidth } = this.systemInfo
      rect.topGap = rect.top - statusBarHeight
      rect.rightGap = screenWidth - rect.right
      return rect
    },

    // 设置自定义导航栏
    setCustomNavBar() {
      if (this.height) return
      const { windowHeight, screenHeight } = this.systemInfo
      this.customNavBar = windowHeight === screenHeight
      if (!this.customNavBar) return
      // 开启自定义导航栏，设置自定义导航的style
      const { statusBarHeight } = this.systemInfo
      this.menuRect = this.getMenuButtonRect()
      const { height, topGap } = this.menuRect
      this.statusBarHeight = statusBarHeight
      this.titleBarHeight = height + topGap * 2
    },

    // 设置安全距离高度
    async setSafeHeight() {
      const safeBottmRect = await this.getDomRect('.safe-bottom-ghost')
      this.safeBottmHeight = safeBottmRect.height
    },

    // 设置占位区域
    async setPlacehold() {
      const { windowHeight, screenHeight } = this.systemInfo
      const [headerRect, footerRect, containerRect] = await Promise.all([this.getDomRect('.header'), this.getDomRect('.footer'), this.getDomRect('.container')])
      this.headerPlaceholderHeight = headerRect.height
      this.footerPlaceholderHeight = footerRect.height
      const needSafeBottom = containerRect.bottom === windowHeight
      if (needSafeBottom && this.footerPlaceholderHeight > 0) {
        const safeBottmRect = await this.getDomRect('.safe-bottom-ghost')
        this.safeBottmHeight = safeBottmRect.height
      }
      this.headerTop = containerRect.top + this.isH5 ? screenHeight - windowHeight : 0
      this.footerTop = containerRect.top + containerRect.height - footerRect.height - this.safeBottmHeight + (this.isH5 ? screenHeight - windowHeight : 0)
    },

    // 主动监听头部底部内容变化，目前仅支持H5
    // async addContentResizeObserver(ins, bottom) {
    //   await this.$nextTick()
    //   // h5模式下换成原生api实现
    //   if (this.isH5) {
    //     ins = new ResizeObserver(async (e) => {
    //       const [headerRect, footerRect] = await Promise.all([this.getDomRect('.header'), this.getDomRect('.footer')])
    //       this.headerPlaceholderHeight = headerRect.height
    //       this.footerPlaceholderHeight = footerRect.height
    //     })
    //     ins.observe(this.$refs.header.$el)
    //     ins.observe(this.$refs.footer.$el)
    //     return
    //   }
    // },

    // 返回方法
    back() {
      if (this.customBack) return this.$emit('back')
      const pages = getCurrentPages()
      if (pages.length === 1) return uni.navigateBack()
      if (this.homeUrl) uni.reLaunch({ url: this.homeUrl })
    },

    // 回流方法，用于异步头部底部内容调用
    reflow() {
      return this.setPlacehold()
    },
  },
  created() {
    this.systemInfo = uni.getSystemInfoSync()
    console.log(this.systemInfo)
    this.setCustomNavBar()

    this.isMp &&
      uni.setBackgroundColor({
        backgroundColor: this.background,
        backgroundColorTop: this.background,
        backgroundColorBottom: this.background,
      })
  },
  async mounted() {
    this.loading = true
    await this.$nextTick()
    await this.setPlacehold()
    await this.$nextTick()
    this.loading = false
  },
  destroyed() {
    resizeObserver && resizeObserver.disconnect()
  },
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
}
.header {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  overflow: hidden;
  z-index: 1;
}

.footer {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  overflow: hidden;
  z-index: 1;
}

.safe-bottom {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: constant(safe-area-inset-bottom);
  /* 兼容 iOS < 11.2 */
  height: env(safe-area-inset-bottom);
}

.safe-bottom-ghost {
  height: constant(safe-area-inset-bottom);
  /* 兼容 iOS < 11.2 */
  height: env(safe-area-inset-bottom);
  position: fixed;
  z-index: -1;
  opacity: 0;
  pointer-events: none;
}

.content {
  flex: 1;
  overflow: auto;
}

.nav-bar {
  background: #fff;
}

.title-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 34rpx;
}

.title-bar-content {
  flex: 1;
  text-align: center;
  min-width: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.title-bar-content-title {
  min-width: 0;
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.back-icon-container {
  display: flex;
  align-items: center;
}
</style>
