### vue.config.js 功能配置
#### 一、打包自动压缩文件
```js
const FileManagerPlugin = require('filemanager-webpack-plugin')
// 打包日期
const YMD = new Date().toLocaleDateString().replace(/\//g, '')
// 打包 package.json 版本
const version = JSON.parse(fs.readFileSync(path.join('', 'package.json')).toString()).version

module.exports = {
  configureWebpack: config => ({
    plugins: [
      new FileManagerPlugin({
        events: {
            onEnd: {
                delete: ['./dist/*.zip'],
                archive: [
                    {source: './dist', destination: `./dist/Webapp-V${version}.${YMD}.zip`},
                ]
            }
        }
      })
    ]
  })
}
```

#### 二、项目对应层级的文件替换依赖包文件
```js
module.exports = {
  configureWebpack: config => ({
    plugins: [
      // 项目里 src/olokin/core 替换掉依赖 @olokin/core 里面的文件
      new webpack.NormalModuleReplacementPlugin(/@olokin[\\|/]/, function (
        resource
      ) {
        if (resource.resource) {
          const target = resource.resource
            .replace('node_modules', 'src')
            .replace('@', '')
          if (fs.existsSync(target)) {
            resource.resource = target
          }
        }
      }),
    ]
  })
}
```