# 简介
用于uniapp框架的页面容器组件。

- 无需额外设置，开箱即用的固定头部和底部，解决ios回弹效果导致头部或底部不固定的问题。
- 自动设置底部安全距离，支持单独设置安全区背景色。
- 自动识别自定义导航栏模式，并提供一套开箱即用的自定义导航栏，支持自定义配置和插槽。

# 组件效果预览
你有两种种方式预览
- 使用浏览器访问该地址：https://dimple-uni-container.vercel.app 进行预览（pc端注意开启开发者模式调整到移动设备）。
- 这个项目本身就是一个 uniapp 项目，git clone该项目到本地后，可使用HBuiderX导入该项目进行预览。

# 目录

- [简介](#简介)
- [组件效果预览](#组件效果预览)
- [目录](#目录)
- [安装](#安装)
- [使用说明](#使用说明)
    - [参数](#参数)
    - [插槽](#插槽)
    - [事件](#事件)
- [开发说明](#开发说明)
- [贡献](#贡献)
    - [赞助](#赞助)
- [许可证](#许可证)
- [最后](#最后)

# 安装

[(Back to top)](#目录)

```
// 在命令行里执行
$ npm i @dimple-smile/uni-container

// 在你的代码里写
import DimpleContainer from '@dimple-smile/uni-container'
```

# 使用说明

[(Back to top)](#目录)

下面是一个最简单的使用例子

> 在src/pages/index/index.vue文件里能看到预览使用的栗子

```
<template>
  <dimple-container>

    <template #header>
      <view class="header"> header </view>
    </template>

    <view class="content"> content </view>

    <template #footer>
      <view class="footer"> footer </view>
    </template>

  </dimple-container>
</template>

<script>
import DimpleContainer from '@dimple-smile/uni-container'

export default {
  components: { DimpleContainer },
}
</script>

<style scoped>
/* css请按需编写 */
</style>

```

## 参数

[(Back to top)](#目录)
| 参数名 | <img width="180px" /> 意义 <img width="180px" /> | 类型 | <img width="120px" /> 默认值 <img width="120px" /> | 说明 |
| -| -| -| - | - |
| height | 容器的高度  | String | '' | 默认占满整个window，如需自定义需要px或%，内部使用flex布局，无需担心高度计算问题 |
| background | 容器的背景色 | String | #eeeeee | 无 |
| footerBackground | 底部的背景色 | String | #ffffff | 安全距离占位区会使用这个背景色 |
| title | 自定义导航栏的标题 | String | '' | 只有开启了自定义导航栏才会生效 |
| navBarStyle | 自定义导航栏的样式 | String | '' | 只有开启了自定义导航栏才会生效 |
| statusBarStyle | 自定义导航栏的状态栏样式 | String | '' | 只有开启了自定义导航栏才会生效 |
| titleBarStyle | 自定义导航栏的标题栏样式 | String | '' | 只有开启了自定义导航栏才会生效 |
| customBack | 开启自定义监听返回事件的标识 | Boolean | false | 只有开启了自定义导航栏才会生效 |
| homeUrl | 返回首页的地址 | String | /pages/index/index | 开启自定义导航栏后，点击返回，页面栈长度为1时会尝试返回首页 |
| backIcon | 返回图标路径 | String | base64 | 默认是官方返回icon的base64字符串，因为是字符串，另外可以传入网络地址 |
| backIconStyle | 返回图标的样式 | String | height: 17px;width: 9px; | 默认使用官方sketch资源里的宽高 |

### 插槽

[(Back to top)](#目录)

| 参数名 | <img width="200px" /> 意义 <img width="200px" /> | 说明 |
| - | - | - |
| 默认 | 用来放置容器内容 | 无
| header | 顶部内容插槽 | 固定在容器顶部，在导航栏之下。如果需要异步设置内容，需要主动调用组件内部的reflow()方法 |
| footer | 底部内容插槽 | 固定在容器底部，在安全距离之上。如果需要异步设置内容，需要主动调用组件内部的reflow()方法 |
| titleLeft | 自定义导航栏内标题栏左侧的内容插槽 | 需要开启自定义导航栏设置才能生效 |
| titleContent | 自定义导航栏内标题栏内容区的插槽 | 需要开启自定义导航栏设置才能生效 |
| titleRight | 自定义导航栏内标题栏右侧的内容插槽 | 需要开启自定义导航栏设置才能生效 |

### 事件

[(Back to top)](#目录)
| 参数名 | <img width="200px" /> 意义 <img width="200px" /> | 说明 |
| - | - | - |
| back | 点击返回按钮发射的事件 | 需要开启customBack选项才能生效 |

# 开发说明

[(Back to top)](#目录)

src文件夹存放着组件的全部源码。入口为src/components/dimple-uni-container.vue。

# 贡献
[(Back to top)](#目录)

@dimple-smile

### 赞助
[(Back to top)](#目录)

Love
# 许可证
[(Back to top)](#目录)

只要不商用，注明出处即可。

[GNU General Public License version 3](https://opensource.org/licenses/GPL-3.0)

# 最后
[(Back to top)](#目录)

谢谢你的使用~
