### 页面使用自适应之后，里面的固定值需要通过函数转换一下
安装自适应依赖
```js
npm i -S postcss-px2rem-exclude
```

在 vue.config.js 配置对应的选项
```js
// vue.config.js
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

新建 src/store 目录并在其下面创建 index.ts，导出 store
```js
// src/store/index.ts

const store = createPinia()

export default store
```

在 main.ts 中引入并使用
```js
// src/main.ts

import store from '@/store'

app.use(store)
```

在 src/store 下面创建一个rem.ts
```js
// src/store/rem.ts

export const useRemStore = defineStore({
  id: 'rem',
  state: () => {
    return {
      width: document.documentElement.clientWidth,
      remUnit: Number(process.env.VUE_APP_BASE_REM_UNIT)
    }
  },
  actions: {
    setRem() {
      const baseSize = this.remUnit
      const width = document.documentElement.clientWidth
      const scale = width / baseSize
      // 设置页面根节点字体大小, 最高放大比例为2
      const size = baseSize * Math.min(scale, 2)
  
      document.documentElement.style.fontSize = size + 'px'
      this.width = width
    },
  },
  getters: {
    getRelationPx(state) {
      const baseSize = state.remUnit
      const scale = state.width / baseSize

      return (px:number) => {
        const size = baseSize * Math.min(scale, 2)
        return (px / baseSize) * size
      }
    }
  }
})
```

在 src/utils 下面创建一个rem.ts
```js
// src/utils/rem.ts

import { useRemStore } from '@/store/rem'

const remStore = useRemStore()

// 设置 html fontsize，被保存当前宽度到 pinia
export function setRem() {
  remStore.setRem()
}

// 返回根据 html fontsize 转换后的值
export function getRelationPx(px:number) {
  return remStore.getRelationPx(px)
}
```

在 main.ts 中引入并使用
```js
// src/main.ts

import { setRem } from '@/utils/rem'

// 页面渲染完毕后，获取宽度并设置
document.addEventListener('DOMContentLoaded', setRem, false)

// 窗口发生改变后，获取宽度并设置
window.addEventListener('resize', setRem, false)
```

在具体页面中使用
```js
  import { getRelationPx } from '@/utils/rem'

  getRelationPx(100)
```