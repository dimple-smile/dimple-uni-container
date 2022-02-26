<template>
  <d-container>
    <template #header>
      <view class="header">
        header
      </view>
    </template>
    <view class="content"> content </view>
    <template #footer>
      <view class="footer"> footer </view>
    </template>
  </d-container>
</template>

<script>
// import DContainer from '@dimple-smile/uni-container'
import DContainer from '../../components/dimple-uni-container.vue'

export default {
  components: { DContainer },
  data() {
    return {}
  },
  methods: {},
}
</script>

<style scoped>
.header {
  padding: 20px;
  background: #ffffff;
}

.content {
  padding: 20px;
}

.scroll-content {
  padding: 20px;
}

.footer {
  padding: 20px;
  background: #ffffff;
}
</style>
