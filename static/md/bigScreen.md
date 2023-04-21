### 大屏可视化配置
一、插件配置
```js
npm i -S postcss-px2rem-exclude

module.exports = {
  css: {
    loaderOptions: {
      postcss: {
        plugins: [
          require('postcss-px2rem-exclude')({
            remUnit: 1920,
            // 使用相对路径 /xxx/xxx，苹果 和 win 的路径不兼容
            exclude: 'BigScreen' //大屏禁止转rem单位
          })
        ]
      }
    }
  }
}
```

二、另外一个插件的配置
```js
npm i -S postcss-pxtorem


css: {
  loaderOptions: {
    postcss: {
      plugins: [
        require('postcss-pxtorem')({
          rootValue: 1920,
          minPixelValue: 2,
          propList: ['*'],
          // 过滤掉.big-screen-开头的 class，不进行 rem 转换
          selectorBlackList: ['.big-screen-']
        })
      ]
    }
  }
}
```

html 页面设置样式
```html
<body class="big-screen-body">
  <div class="big-screen-main" :style="style"></div>
</body>
```

js 代码
```js
import { isFullScreenState } from '@/utils/base'

resetZoom()
window.addEventListener('resize', resetZoom, false)

function resetZoom() {
  const body = document.documentElement
  const wScale = body.clientWidth / 1920
  const hScale = body.clientHeight / 1080

  if (hScale > wScale) {
    this.style = `transform: scale3d(${wScale}, 1, 1) translateX(-50%)`
  } else {
    this.style = `transform: scale3d(1, ${hScale}, 1) translateX(-50%)`
  }
}

// 大屏全屏方案，按 F11、ESC、刷新页面，退出全屏且跳转其他页面
setTimeout(() => {
  // 不是全屏状态，跳到其他页面
  if (!isFullScreenState()) {
    const flag = Math.ceil(window.devicePixelRatio * document.body.offsetHeight) === 1080

    // F11 状态下刷新，会识别为非全屏
    if (!flag) {
      this.$router.push({ path: '/xxx' })
    }
  }
}, 0);

window.addEventListener('fullscreenchange', this.exitFullScreen, false)
// 播放中视频全屏不能跳转，退出视频即可
const isNoExit = false

function exitFullScreen() {
  setTimeout(() => {
    const videos = this.$children[0]

    if (videos && videos.$refs.videoGrid && videos.$refs.videoGrid.isFullScreen) {
      isNoExit = true
    } else {
      if (!isNoExit) {
        this.$router.push({ path: '/xxx' })
      }
    }
  }, 30);
}
```

css 样式
```css
.big-screen-body {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 像素单元声明中使用大写 px -> Px，不进行 rem 转换 */
.big-screen-main {
  width: 1920Px;
  height: 1080Px;
  transform-origin: left top;
  box-sizing: border-box;
  overflow: hidden;
}
```