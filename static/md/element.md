### element 一些用法配置
#### 重置 message，防止重复点击重复弹出 message 弹框
```js
import { message } from '@/utils/resetMessage'

// 重写 message 提示框一定要放在 app.use(ElementPlus) 之后
app.config.globalProperties.$msg = message

// 引入 getCurrentInstance, 通过 proxy.$msg 来调用
const { proxy } = getCurrentInstance()
proxy.$msg.success('提示')
```

#### 全屏模式下，messageBox 提示无法显示
```js
import { resetMessageTips } from '@/utils/element'

resetMessageTips(this.$el)
```

#### 全局 z-index 数值管理
```js
import PopupManager from 'element-ui/lib/utils/popup/popup-manager'

zIndex() {
  return PopupManager.nextZIndex()
}
```

#### 日期选择器禁用
```js
import { getTodayLastTime } from '@/utils/date'

const pickerOptions = {
  disabledDate: (time) => {
    const lasttime = new Date().getTime() + getTodayLastTime()

    if (new Date(time).getTime() > lasttime) {
      return time.getTime() > lasttime
    }
  },
}
```

#### 表格行高亮
```js
import { setHighlightRow } from '@/utils/element'

setHighlightRow()
```

#### 表格列标题渲染
```js
import { renderLabelFn } from '@/utils/element'

renderLabelFn()
```