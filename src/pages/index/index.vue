<template>
  <d-container title="dimple-uni-container" :mutation="mutation">
    <template #header>
      <view class="header" v-if="headerVisible"> header </view>
    </template>
    <d-scroll :total="total" :skip="skip" :limit="limit" @fetch="fetch">
      <view class="scroll-content">
        <view v-for="(item, index) in list" :key="index" class="scroll-item">{{ item + 1 }}</view>
      </view>
    </d-scroll>
    <template #footer>
      <view>
        <view class="footer" v-if="footerVisible">footer</view>
      </view>
    </template>
  </d-container>
</template>

<script>
import DContainer from '../../components/dimple-uni-container.vue'
import DScroll from '@dimple-smile/uni-scroll'
export default {
  components: { DContainer, DScroll },
  data() {
    return {
      list: [],
      total: -1,
      skip: -1,
      limit: 20,
      headerVisible: false,
      footerVisible: false,
    }
  },
  computed: {
    mutation() {
      return {
        headerVisible: this.headerVisible,
        footerVisible: this.footerVisible,
      }
    },
  },
  methods: {
    async fetch(e) {
      uni.showLoading({ title: '加载中' })
      const { stop } = e
      try {
        await this.getData(e)
      } catch {}
      stop()
      uni.hideLoading()
    },
    async getData(options = {}) {
      const { page = 1, skip = 0, limit = this.limit } = options
      const res = await new Promise((res) =>
        setTimeout(() => {
          const data = Array(limit)
            .fill('')
            .map((item, index) => (page === 1 ? index : skip + index))
          res(data)
        }, 2000)
      )
      this.total = 100
      if (page === 1) this.list = []
      this.list.push(...res)
      this.skip = this.list.length
    },
  },
  async mounted() {
    uni.showLoading({ title: '加载中' })
    await this.getData()
    uni.hideLoading()
    setTimeout(() => {
      this.headerVisible = true
      setTimeout(() => {
        this.footerVisible = true
      }, 2000)
    }, 2000)
  },
}
</script>

<style scoped>
.header {
  padding: 20px;
  background: #ffffff;
}

.content {
  flex: 1;
  overflow: auto;
}

.scroll-content {
  padding: 20px;
}

.scroll-item {
  background: #ffffff;
  border-radius: 2px;
  padding: 20px;
  margin-bottom: 20px;
}

.footer {
  padding: 20px;
  background: #ffffff;
}
</style>
