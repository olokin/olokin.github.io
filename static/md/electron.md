#### electron 配置方法
[electron-builder 快速打包全平台应用](https://mp.weixin.qq.com/s?__biz=MzUxOTA2MDAyNQ==&mid=2247484928&idx=1&sn=6c74fcf8c92ea4ccd949a7c3c9990a32&chksm=f9fe2956ce89a0403d46cc98da241029ce261a26fa9703fb5f4ccc9c472809df4ba6c659357a&scene=27)


package.json 配置
```js
{
  "scripts": {
    "electron:build": "vue-cli-service electron:build",
    "electron:serve": "vue-cli-service electron:serve",
    "postinstall": "electron-builder install-app-deps",
    "postuninstall": "electron-builder install-app-deps",
    "electron:icons": "electron-icon-builder --input=./public/icon.png --output=build --flatten"
  },
  "main": "background.js",
  "dependencies": {
    "electron-icon-builder": "2.0.1",
  },
  "devDependencies": {
    "electron": "13.1.7",
    "electron-is-dev": "2.0.0",
    "electron-log": "4.4.0",
    "vue-cli-plugin-electron-builder": "2.1.1"
  },
}

```

vue.config.js 配置
```js
module.exports = {
  configureWebpack: config => ({
    entry: 'jxudp-root/jxudp-core/main.js',
  }),
  pluginOptions: {
    electronBuilder: {
      disableMainProcessTypescript: false,
      mainProcessTypeChecking: false,
      builderOptions: {
        appId: 'com.olo.app',
        productName: '监控平台',
        copyright: '出品公司：xxx股份有线公司',
        win: {
          icon: 'build/icons/icon.ico',
        },
        nsis: {
          oneClick: false,
          perMachine: false,
          allowElevation: true,
          allowToChangeInstallationDirectory: true,
          installerIcon: './build/icons/icon.ico',
          uninstallerIcon: './build/icons/icon.ico',
          installerHeaderIcon: './build/icons/icon.ico',
          createDesktopShortcut: true,
          createStartMenuShortcut: true,
          runAfterFinish: true,
          shortcutName: '监控平台',
        },
      },
    },
  },
}

```


background.js
```js
// 页面是固定写死的，有部分内部超出不显示，设置缩放比例
app.commandLine.appendSwitch('high-dpi-support', 'true')
app.commandLine.appendSwitch('force-device-scale-factor', '1')

// 加载不安全的https网址
app.commandLine.appendSwitch('--ignore-certificate-errors', 'true')

// 禁用缓存
app.commandLine.appendSwitch('--disable-http-cache')
```

createWindow.ts
```js
import { BrowserWindow, Menu } from 'electron'

export function loginWindow() {
  // 设置顶部菜单栏
  const clearObj = {
    storages: ['appcache', 'filesystem', 'indexdb', 'localstorage', 'shadercache', 'websql', 'serviceworkers', 'cachestorage'],
  }
  let menuTemplate = [
    {
        label: '设置',
        submenu:[
          {
            label: '清除缓存数据',
            click: (item, focusedWindow) => {
              if (focusedWindow) {
                // 重载之后, 刷新并关闭所有之前打开的次要窗体
                if (focusedWindow.id === 1) {
                  BrowserWindow.getAllWindows().forEach((win) => {
                    if (win.id > 1) win.close()
                  });
                }
                focusedWindow.webContents.session.clearStorageData(clearObj)
                focusedWindow.webContents.reloadIgnoringCache()
                focusedWindow.reload()
              }
            }
          }
        ]
    }
  ]

  let menuBuilder = Menu.buildFromTemplate(menuTemplate)
  Menu.setApplicationMenu(menuBuilder)
}

```

communication.js
```js
// 与渲染进程通信
import { ipcMain, screen } from 'electron'

export function communication(win) {
  ipcMain.on('DOMLOADED', (event, arg) => {
    // 主进程获取屏幕分辨率，通知渲染进程执行 workAreaSize 函数
    win.webContents.send('workAreaSize', screen.getPrimaryDisplay().workAreaSize.width)
  })
}
```

app.vue
```js
// 渲染进程 DOMContentLoaded 执行完毕后，重新根据屏幕分辨率调整缩放比例
if (window.require && window.require('electron')) {
  // 通知主进程执行 DOMLOADED 函数
  window.require('electron').ipcRenderer.send('DOMLOADED', true)
  // 渲染进程重新设置分辨率
  window.require('electron').ipcRenderer.on('workAreaSize', (ev, data) => {
    window.require('electron').webFrame.setZoomFactor(parseFloat(data / 1920))
  })
}
```