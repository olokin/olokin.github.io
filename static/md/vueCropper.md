### vue-cropper 用法及坑
```html
<vueCropper
  ref="cropper"
  :img="imgOption.img"
  :outputSize="imgOption.size"
  :outputType="imgOption.outputType"
  :info="true"
  :full="imgOption.full"
  :canMove="imgOption.canMove"
  :canMoveBox="imgOption.canMoveBox"
  :fixedBox="imgOption.fixedBox"
  :original="imgOption.original"
  :autoCrop="imgOption.autoCrop"
  :autoCropWidth="imgOption.autoCropWidth"
  :autoCropHeight="imgOption.autoCropHeight"
  :centerBox="imgOption.centerBox"
  :high="imgOption.high"
  :infoTrue="imgOption.infoTrue"
  :enlarge="imgOption.enlarge">
</vueCropper>

<div class="dialog-footer cropper_btn">
  <el-button  @click="clearFilet()">取消</el-button>
  <el-button type="primary" @click="finish('blob')">确定</el-button>
</div>
```

#### 裁剪出来的图比原图还大，导致图片显示一部分，剩下的是空白
```js
// 重置 cropper 块大小，截取的图片可能比原图还大
this.$refs.uploader.uploader.fileList[0].chunks[0].endByte = data.size
```

```js
export default {
  data() {
    return {
      previewImgURL: '', // 预览图片
      imgOption: {
        // vue-cropper 参数配置
        img: '', // 裁剪图片的地址
        size: 1, // 裁剪生成图片的质量
        full: false, // 是否输出原图比例的截图
        // outputType: 'png', //输出图片格式
        canMove: true, //能否拖动图片
        fixedBox: true, //截图框固定大小
        original: false, //否显示原始宽高
        canMoveBox: true, //能否拖动截图框
        autoCrop: true, //autoCrop 是否自动生成截图框
        // 只有自动截图开启 宽度高度才生效
        autoCropWidth: 290,
        autoCropHeight: 160,
        centerBox: false, //截图框是否限制在图片里
        high: true, //是否根据dpr生成适合屏幕的高清图片
      },
    }
  },
  methods: {
    //图片裁剪后替换原图片
    finish(type) {
      if (type === 'blob') {
        this.$refs.cropper.getCropBlob((data) => {
          if (data.size > 2048000) {
            this.$message.warning('只允许上传2M以内的文件')
            this.clearFilet()
          } else {
            this.$refs.uploader.uploader.fileList[0].file = data
            this.$refs.uploader.uploader.files[0].file = data
            this.$refs.uploader.uploader.fileList[0].size = data.size
            this.$refs.uploader.uploader.files[0].size = data.size
            this.$refs.uploader.uploader.fileList[0].uniqueIdentifier = data.size + '-' + this.$refs.uploader.uploader.fileList[0].name

            // 重置 cropper 块大小，截取的图片可能比原图还大
            this.$refs.uploader.uploader.fileList[0].chunks[0].endByte = data.size
            this.uploadInfo.file_size = data.size
            this.previewImgURL = window.URL.createObjectURL(data)
          }
        })
      } else if (type === 'base64') {
        this.$refs.cropper.getCropData((data) => {
          let modelView = this.dataURLtoFile(data, this.$refs.uploader.uploader.fileList[0].name)

          if (modelView.size > 2048000) {
            this.$message.warning('只允许上传2M以内的文件')
            this.clearFilet()
          } else {
            this.$refs.uploader.uploader.fileList[0].file = modelView
            this.$refs.uploader.uploader.files[0].file = modelView
            this.$refs.uploader.uploader.fileList[0].size = modelView.size
            this.$refs.uploader.uploader.files[0].size = modelView.size

            this.previewImgURL = data
            this.uploadInfo.file_size = this.$refs.uploader.uploader.fileList[0].size
          }
        })
      }
    },
    // dataUrl转为file
    dataURLtoFile(dataUrl, fileName) {
      const arr = dataUrl.split(',')
      const mime = arr[0].match(/:(.*?);/)[1]
      const bstr = window.atob(arr[1])
      const n = bstr.length
      const u8arr = new Uint8Array(n)

      while(n--){
        u8arr[n] = bstr.charCodeAt(n)
      }

      return new File([u8arr], fileName, {type:mime})
    },
    clearFilet() {},
  },
}
```