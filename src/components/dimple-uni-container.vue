<template>
  <view v-if="!loading" ref="container" class="container" :style="{ height: computedHeight, background: background }">
    <!-- header -->
    <view class="header" :style="[headerFixedStyle]">
      <!-- 自定义nav-bar -->
      <view v-if="isCustomNavBar" class="nav-bar" :style="navBarStyle">
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

    <!-- header placeholder-->
    <view class="header-placeholder" :style="[headerPlaceholderStyle]"> </view>

    <!-- 内容区 -->
    <view ref="content" class="content">
      <!-- 默认slot -->
      <view class="content-slot">
        <slot></slot>
      </view>
    </view>

    <!-- footer placeholder-->
    <view class="footer-placeholder" :style="[footerPlaceholderStyle]"></view>

    <!-- footer -->
    <view ref="footer" class="footer" :style="[footerFixedStyle]">
      <div class="footer-slot">
        <slot name="footer"></slot>
      </div>
      <view v-if="safeBottmVisible" ref="safeBottom" class="safe-bottm" :style="{ background: footerBackground }"></view>
    </view>
  </view>
</template>

<script>
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
    mutation: {}, // 突变数据，该数据变化会触发布局变化

    customNavBar: {}, // 是否开启自定义头
  },
  data() {
    return {
      systemInfo: {}, // 记录系统信息
      loading: false, // 加载标识

      headerFixedStyle: {},
      footerFixedStyle: {},

      headerPlaceholderStyle: {},
      footerPlaceholderStyle: {},

      isCustomNavBar: false, // 是否开启自定义navbar的标识
      statusBarHeight: 0, // 自定义navbar状态栏高度
      titleBarHeight: 0, // 自定义navbar标题栏高度
      menuRect: {}, // 小程序胶囊按钮的rect信息

      safeBottmVisible: true, // 是否显示安全距离
    }
  },
  computed: {
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
    computedHeight() {
      if (this.height) return this.height
      const { windowTop } = this.systemInfo
      if (this.isH5) return `calc(100vh - ${windowTop}px)`
      return '100vh'
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
  },
  watch: {
    mutation: {
      handler() {
        this.$nextTick(() => this.setStyle())
      },
      deep: true,
    },
  },
  methods: {
    // 根据classname获取dom元素clientRect信息，并把方法promise化
    getDomRect(className) {
      // const query = uni.createSelectorQuery().in(this)
      return new Promise((res) => {
        uni.getSystemInfo({
          success: () => {
            let info = uni.createSelectorQuery().in(this).select(className)
            info
              .boundingClientRect((data) => {
                res(data)
              })
              .exec()
          },
        })
      })
    },

    // 获取胶囊Rect信息，兼容写法
    getMenuButtonRect() {
      if (this.isH5 || this.isApp) return { height: 44, width: 90, rightGap: 10 }
      const rect = uni.getMenuButtonBoundingClientRect()
      const { screenWidth } = this.systemInfo
      rect.rightGap = screenWidth - rect.right
      rect.height = this.isIos ? 44 : 48
      return rect
    },

    // 设置自定义导航栏
    setCustomNavBar() {
      if (this.customNavBar === false) return
      if (this.height) return
      const { windowTop, screenHeight, windowHeight } = this.systemInfo
      console.log(this.systemInfo)
      let isCustomNavBar = false
      if (this.isMp || this.isApp) isCustomNavBar = screenHeight === windowHeight
      if (this.isH5) isCustomNavBar = windowTop === 0
      if (this.customNavBar) isCustomNavBar = true
      this.isCustomNavBar = isCustomNavBar
      if (!isCustomNavBar) return
      // 开启自定义导航栏，设置自定义导航的style
      const { statusBarHeight } = this.systemInfo
      this.menuRect = this.getMenuButtonRect()
      const { height } = this.menuRect
      this.statusBarHeight = statusBarHeight
      this.titleBarHeight = height
    },

    // 设置占位区域
    async setStyle() {
      this.safeBottmVisible = true
      this.headerFixedStyle = {}
      this.footerFixedStyle = {}
      this.headerPlaceholderStyle = {}
      this.footerPlaceholderStyle = {}

      await this.$nextTick()

      let containerRect = {}
      let contentRect = {}
      let headerHeight = 0
      let footerHeight = 0

      if (this.isMp || this.isApp) {
        const [containerRectRes, contentRectRes] = await Promise.all([this.getDomRect('.container'), this.getDomRect('.content')])
        containerRect = containerRectRes
        contentRect = contentRectRes
        footerHeight = containerRect.height - containerRect.top - contentRect.top - contentRect.height
      }

      if (this.isH5) {
        containerRect = this.$refs.container.$el.getBoundingClientRect()
        contentRect = this.$refs.content.$el.getBoundingClientRect()
        const footerRect = this.$refs.footer.$el.getBoundingClientRect()
        const safeBottomRect = this.$refs.safeBottom.$el.getBoundingClientRect()
        footerHeight = footerRect.height
        if (footerRect.height === safeBottomRect.height) footerHeight = 0
      }

      headerHeight = contentRect.top - containerRect.top
      if (footerHeight === 0) this.safeBottmVisible = false

      this.headerPlaceholderStyle = { height: headerHeight + 'px', background: this.background }
      this.headerFixedStyle = { position: 'fixed', top: contentRect.top - headerHeight + 'px', zIndex: 1 }

      this.footerPlaceholderStyle = { height: footerHeight + 'px', background: this.background }
      this.footerFixedStyle = { position: 'fixed', top: contentRect.top + contentRect.height + 'px', zIndex: 1 }
    },

    // 返回方法
    back() {
      if (this.customBack) return this.$emit('back')
      const pages = getCurrentPages()
      if (pages.length > 1) return uni.navigateBack()
      if (this.homeUrl) uni.reLaunch({ url: this.homeUrl })
    },
  },
  created() {
    this.systemInfo = uni.getSystemInfoSync()
    this.setCustomNavBar()
  },
  mounted() {
    this.setStyle()
  },
}
</script>

<style scoped>
.container {
  width: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.header {
  width: 100%;
}

.footer {
  width: 100%;
}

.safe-bottm {
  height: calc(env(safe-area-inset-bottom) / 2);
}

.content {
  flex: 1;
  overflow: hidden;
}

.content-slot {
  height: 100%;
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
