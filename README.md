# vue-CutImage
一个用于本地头像剪切的vue组件, 其实稍加改造就和原生的一模一样, 无非就是变量函数方法的作用域
## 使用的技术
1. canvas的截取
2. drag H5的拖动
3. FileReader读取拖进的图片文件
4. 其他就是一些逻辑处理了
## 使用教程(其实就是一个组件,所以使用起来就三部)
1. 在自己的父组件中引入这个组件
2. 声明并使用
3. 提交时在提交的方法中写自己所需要提交的形式
## 看一下源码,大部分都已经注释了
```
<template>
  <div class="cut-board">
    <div class="left-cut-board">
      <div class="img-board" @mouseenter="listenMousewheel" @mouseleave="removeMousewheel" @dragover="allowDrop">
        <canvas id="canvas-clip"></canvas>
      </div>
      <div class="img-option">
        <div class="upload-button">
          <span class="upload-img" @click="focusInput">上传</span>
          <input id="taggle-img" type="file" accept="image/*" @change="uploadSuccess">
        </div>
        <div class="control-size">
          <span>~</span>
          <div class="slide-line">
            <div class="slide-block"></div>
          </div>
          <span>+</span>
        </div>
      </div>
    </div>
    <div class="right-show-board">
      <div class="close-cut-board"></div>
      <div class="show-board">
        <canvas id="right-canvas" width="400" height="400">你的浏览器过于老旧无法实现canvas</canvas>
      </div>
      <div class="submit-buttons">
        <span class="submit-img" @click="upClipImg">提交</span>
      </div>
    </div>
  </div>
</template>

<script type="text/javascript">
export default {
  name: 'CutImage',
  data () {
    return {
      theImage: '', // 储存来自不同地方的图片
      theImg: '', // 储存上传的图片这个对象
      fromDragImg: '', // 储存来自拖动来的图片
      fromDrag: false, // 是否来自拖动来的图片
      isCanvasShow: false, // 让canvas动态赋值一次宽高
      isHaveImg: false, // 是否存在有图片
      isMoveInCanvas: false, // 鼠标是否移入canvas
      isMousedown: false, // canvas 判断鼠标是否按下
      isSlideMousedown: false, // 滑块判断鼠标是否按下
      imgWH: 0, // 图片的长宽比(用于第一次显示的长宽)
      mouseX: 0, // 鼠标按下时的横坐标
      mouseY: 0, // 鼠标按下时的纵坐标
      imgMoveLeft: 0, // canvas 图片放置的横坐标位置(从哪开始画)
      imgMoveTop: 0, // canvas 图片放置的纵坐标的位置
      imgMoveX: 0, // canvas 记录图片在鼠标抬起时图片的横坐标位置
      imgMoveY: 0, // canvas 记录图片在鼠标抬起时图片的纵坐标位置
      slideFirstX: 0, // 鼠标按下时滑块的横坐标
      slideFinalX: 0, // 鼠标抬起时记录的横坐标
      firstImgWidth: 0, // 记录滑块操作后图片的宽度
      firstImgHeight: 0 // 记录滑块操作后图片的高度
    }
  },
  computed: {
    imgBoard () {
      return document.getElementsByClassName('img-board')[0]
    },
    theCanvas () {
      return document.getElementById('canvas-clip')
    },
    canvasCtx () {
      return this.theCanvas.getContext('2d')
    },
    canvasWidth () {
      return this.imgBoard.getBoundingClientRect().width * 2
    },
    canvasHeight () {
      return this.imgBoard.getBoundingClientRect().height * 2
    },
    canvasWH () {
      return this.canvasWidth / this.canvasHeight
    },
    clipLightX () {
      return (this.canvasWidth - 400) / 2
    },
    clipLightY () {
      return (this.canvasHeight - 400) / 2
    },
    slideLine () {
      return document.getElementsByClassName('slide-line')[0]
    },
    slideLineWidth () {
      return this.slideLine.clientWidth
    },
    slideBlock () {
      return document.getElementsByClassName('slide-block')[0]
    },
    // 划线减去滑块的宽度, 防止划出
    lineBlockWidth () {
      return this.slideLine.clientWidth - this.slideBlock.clientWidth
    },
    rightCanvas () {
      return document.getElementById('right-canvas')
    },
    rightCanvasCtx () {
      return this.rightCanvas.getContext('2d')
    }
  },
  mounted () {
    this.$nextTick(() => {
      // 监听拖进图片
      this.imgBoard.addEventListener('drop', (e) => {
        e.preventDefault()
        this.fromDrag = true
        this.fromDragImg = e.dataTransfer.files[0]
        if (this.fromDragImg.type.indexOf('image/') === 0) {
          this.uploadSuccess()
        } else {
          alert('请上传正确的图片!!!')
        }
      })
      window.addEventListener('mousedown', (e) => {
        // 滑块 鼠标按下时
        if (e.target === this.slideBlock && this.isHaveImg) {
          this.isSlideMousedown = true
          this.mouseX = e.clientX
        }
        // canvas 鼠标按下时
        if (e.target === this.theCanvas && this.isHaveImg) {
          this.isMousedown = true
          this.mouseX = e.clientX
          this.mouseY = e.clientY
        }
      })
      window.addEventListener('click', (e) => {
        // 监听点击滑线的位置
        if (e.target === this.slideLine && this.isHaveImg) {
          let useMinLeft = this.theImg.width
          let useMinTop = this.theImg.height
          let theDistance = e.offsetX - this.slideFirstX
          this.slideFirstX = e.offsetX
          this.slideBlock.style.left = e.offsetX + 'px'
          this.theImg.width = this.firstImgWidth += theDistance
          this.theImg.height = this.theImg.width / this.imgWH
          if (Math.min(this.theImg.width, this.theImg.height) > 400) {
            this.imgMoveLeft = this.imgMoveX -= theDistance / 2 // 同步图片起始位置
            this.imgMoveTop = this.imgMoveY -= theDistance / 2 / this.imgWH
          } else {
            if (this.imgWH > 1) {
              this.imgMoveTop = this.imgMoveY -= (400 - useMinTop) / 2
              this.imgMoveLeft = this.imgMoveX -= (400 - useMinTop) / 2 * this.imgWH
              this.theImg.height = this.firstImgHeight = 400
              this.theImg.width = this.firstImgWidth = 400 * this.imgWH
            } else {
              this.imgMoveLeft = this.imgMoveX -= (400 - useMinLeft) / 2
              this.imgMoveTop = this.imgMoveY -= (400 - useMinLeft) / 2 / this.imgWH
              this.theImg.width = this.firstImgWidth = 400
              this.theImg.height = this.firstImgHeight = 400 / this.imgWH
            }
          }
          this.redrawImage()
        }
      })
      // 监听鼠标的移动
      window.addEventListener('mousemove', (e) => {
        // 监听图片的移动
        if (this.isMousedown) {
          this.imgMoveLeft = this.imgMoveX + (e.clientX - this.mouseX) * 2
          this.imgMoveTop = this.imgMoveY + (e.clientY - this.mouseY) * 2
          // 拖动时的边界问题
          if (this.imgMoveLeft < -this.theImg.width) {
            this.imgMoveLeft = -this.theImg.width
          }
          if (this.imgMoveLeft > this.canvasWidth) {
            this.imgMoveLeft = this.canvasWidth
          }
          if (this.imgMoveTop < -this.theImg.height) {
            this.imgMoveTop = -this.theImg.height
          }
          if (this.imgMoveTop > this.canvasHeight) {
            this.imgMoveTop = this.canvasHeight
          }
          this.redrawImage()
        }
        // 监听滑块的移动
        if (this.isSlideMousedown) {
          // 监听鼠标的横坐标位移
          let slideMoveX = e.clientX - this.mouseX
          this.slideFinalX = this.slideFirstX + slideMoveX
          if (this.slideFinalX < 0) {
            this.slideFinalX = 0
          }
          if (this.slideFinalX > this.lineBlockWidth) {
            this.slideFinalX = this.lineBlockWidth
          }
          this.slideBlock.style.left = this.slideFinalX + 'px'
          this.theImg.width = this.firstImgWidth + slideMoveX
          this.theImg.height = this.theImg.width / this.imgWH
          if (Math.min(this.theImg.width, this.theImg.height) < 400) {
            if (this.imgWH > 1) {
              this.theImg.height = 400
              this.theImg.width = 400 * this.imgWH
            } else {
              this.theImg.width = 400
              this.theImg.height = 400 / this.imgWH
            }
          } else {
            this.imgMoveLeft = this.imgMoveX - slideMoveX / 2 // 同步图片起始位置
            this.imgMoveTop = this.imgMoveY - slideMoveX / 2 / this.imgWH
          }
          this.redrawImage()
        }
      })
      // 监听鼠标抬起
      window.addEventListener('mouseup', () => {
        if (this.isHaveImg) {
          // canvas 鼠标抬起时记录值
          this.isMousedown = false
          this.imgMoveX = this.imgMoveLeft
          this.imgMoveY = this.imgMoveTop
          // 滑块鼠标抬起时记录值
          if (this.isSlideMousedown) {
            this.isSlideMousedown = false
            this.slideFirstX = this.slideFinalX
            this.firstImgWidth = this.theImg.width
            this.firstImgHeight = this.theImg.height
          }
        }
      })
      // 监听鼠标滚轮滚动
      window.addEventListener('mousewheel', (e) => {
        if (this.isMoveInCanvas) {
          e = e || window.event
          e.preventDefault()
          this.theImg.width += e.wheelDelta / 10
          this.theImg.height = this.theImg.width / this.imgWH
          // 图片缩放最小宽高
          if (Math.min(this.theImg.width, this.theImg.height) > 400 || e.wheelDelta > 0) {
            this.imgMoveX = this.imgMoveLeft -= e.wheelDelta / 20 // 同步图片起始位置
            this.imgMoveY = this.imgMoveTop -= e.wheelDelta / 20 / this.imgWH
            this.firstImgWidth = this.theImg.width // 同步滑块的起始图片宽度
          } else {
            this.theImg.width -= e.wheelDelta / 10
            this.theImg.height = this.theImg.width / this.imgWH
            this.firstImgWidth = this.theImg.width // 同步滑块的起始图片宽度
          }
          this.redrawImage()
        }
      }, false)
      // 兼容火狐浏览器
      window.addEventListener('DOMMouseScroll', (e) => {
        if (this.isMoveInCanvas) {
          e = e || window.event
          e.preventDefault()
          this.theImg.width -= e.detail * 4
          this.theImg.height = this.theImg.width / this.imgWH
          if (Math.min(this.theImg.width, this.theImg.height) >= 400 || e.detail < 0) {
            this.imgMoveX = this.imgMoveLeft += e.detail * 2 // 同步图片起始位置
            this.imgMoveY = this.imgMoveTop += e.detail * 2 / this.imgWH
            this.firstImgWidth = this.theImg.width // 同步滑块的起始图片宽度
          } else {
            this.theImg.width += e.detail * 4
            this.theImg.height = this.theImg.width / this.imgWH
            this.firstImgWidth = this.theImg.width // 同步滑块的起始图片宽度
          }
          this.redrawImage()
        }
      }, false)
    })
  },
  methods: {
    // 触发上传时隐藏的input
    focusInput () {
      document.getElementsByClassName('shade-floor')[0].style.display = 'block'
      document.getElementById('taggle-img').click()
    },
    // 允许直接拖动桌面图片上传
    allowDrop (e) {
      e.preventDefault()
    },
    uploadSuccess () {
      // 只赋值一次canvas宽高等于包含元素(这里其实是两倍,用于解决canvas图片不清晰问题)
      if (!this.isCanvasShow) {
        this.isCanvasShow = true
        this.theCanvas.width = this.canvasWidth
        this.theCanvas.height = this.canvasHeight
      }
      // 第一次打开滑块居中显示
      this.slideFirstX = this.slideLineWidth / 2
      this.slideBlock.style.left = this.slideFirstX + 'px'
      // H5的 FileReader 读取上传的图片
      if (this.fromDrag) {
        this.theImage = this.fromDragImg
        this.fromDrag = false
      } else {
        this.theImage = document.getElementById('taggle-img').files[0]
      }
      let imgReader = new FileReader()
      imgReader.readAsDataURL(this.theImage)
      imgReader.onload = (e) => {
        this.theImg = new Image()
        this.theImg.src = e.target.result
        this.theImg.onload = () => {
          // 当更换图片时重置一些东西
          this.isHaveImg = true
          // 及时清空 input 否则换图时点击取消控制台报错
          document.getElementById('taggle-img').value = null
          // 重新计算新图片的长宽比
          this.imgWH = this.theImg.width / this.theImg.height
          // 计算第一次图片的显示大小和位置
          if (this.imgWH >= this.canvasWH) {
            this.theImg.width = this.canvasWidth
            this.theImg.height = this.theImg.width / this.imgWH
            // 重置图片的起始位置及调动时的起始位置
            this.imgMoveX = this.imgMoveLeft = 0
            this.imgMoveY = this.imgMoveTop = (this.canvasHeight - this.theImg.height) / 2
          } else {
            this.theImg.height = this.canvasHeight
            this.theImg.width = this.theImg.height * this.imgWH
            // 重置图片的起始位置及调动时的起始位置
            this.imgMoveX = this.imgMoveLeft = (this.canvasWidth - this.theImg.width) / 2
            this.imgMoveY = this.imgMoveTop = 0
          }
          // 赋值第一次图片拖动时的宽高
          this.firstImgWidth = this.theImg.width
          this.firstImgHeight = this.theImg.height
          this.redrawImage()
        }
      }
    },
    // 准备监听滑轮的事件
    listenMousewheel (e) {
      if (this.isHaveImg) {
        this.isMoveInCanvas = true
      }
    },
    // 移出canvas时移除监听滑轮事件
    removeMousewheel () {
      if (this.isHaveImg) {
        this.isMoveInCanvas = false
      }
    },
    // 每次移动或者操作大小变化时重绘
    redrawImage () {
      // 清屏左边canvas重绘
      this.canvasCtx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.canvasCtx.drawImage(this.theImg, this.imgMoveLeft, this.imgMoveTop, this.theImg.width, this.theImg.height)
      // 画到右边的canvas上
      let imgClipData = this.canvasCtx.getImageData(this.clipLightX, this.clipLightY, 400, 400)
      this.rightCanvasCtx.clearRect(0, 0, 200, 200)
      this.rightCanvasCtx.putImageData(imgClipData, 0, 0)
      // 加上阴影层
      this.canvasCtx.save() // 保存第一个图层画上的图片
      this.canvasCtx.fillStyle = 'rgba(0, 0, 0, 0.4)'
      this.canvasCtx.fillRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.canvasCtx.rect(this.clipLightX, this.clipLightY, 400, 400)
      this.canvasCtx.fillStyle = 'rgba(255, 255, 255, 10)'
      this.canvasCtx.fill()
      this.canvasCtx.restore()
      // 高亮显示中间的窗口
      this.canvasCtx.save()
      this.canvasCtx.beginPath() // 开始一个路径
      this.canvasCtx.rect(this.clipLightX, this.clipLightY, 400, 400) // 这个路径的路线
      this.canvasCtx.closePath() // 关闭这个路径
      this.canvasCtx.clip() // 剪下这块
      this.canvasCtx.drawImage(this.theImg, this.imgMoveLeft, this.imgMoveTop, this.theImg.width, this.theImg.height)
      this.canvasCtx.restore()
    },
    // 提交剪切好的图片
    upClipImg () {
      console.log('此处编辑你获取canvas中截取的数据,根据你的需求进行操作即可')
    }
  }
}
</script>

<style lang="scss" scoped>
.cut-board {
  position: fixed;
  width: 52em;
  height: 26em;
  top: 20%;
  left: 20%;
  z-index: 200;
  border-radius: 10px;
  overflow: hidden;
  background: #fff;
  user-select: none; // 很重要,否则会出现拖动bug
  .left-cut-board {
    float: left;
    width: 30em;
    height: 25em;
    margin: 1em 0 1em 1em;
    .img-board {
      width: 100%;
      height: 80%;
      canvas#canvas-clip {
        width: 100%;
        height: 100%;
        cursor: pointer;
      }
    }
    .img-option {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      height: 20%;
      .upload-button {
        width: 5em;
        height: 2.5em;
        overflow: hidden;
        span.upload-img {
          display: block;
          width: 100%;
          height: 100%;
          line-height: 2.5;
          text-align: center;
          cursor: pointer;
          border-radius: 5px;
          border: 1px solid #ddd;
          transition: background 0.3s;
        }
      }
      .control-size {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 18em;
        height: 2.5em;
        .slide-line {
          position: relative;
          width: 14.5em;
          height: 2px;
          cursor: pointer;
          background: #999;
          .slide-block {
            position: absolute;
            width: 0.5em;
            height: 1em;
            top: -0.5em;
            background: #999;
          }
        }
        span {
          font-size: 1.6em;
          color: #999;
        }
      }
    }
  }
  .right-show-board {
    float: right;
    display: flex;
    flex-flow: column wrap;
    justify-content: space-between;
    width: 20em;
    height: 100%;
    .close-cut-board {
      width: 100%;
      height: 3em;
      text-align: right;
      overflow: hidden;
    }
    .show-board {
      display: flex;
      justify-content: center;
      width: 100%;
      height: 12em;
      canvas#right-canvas {
        border-radius: 50%;
        width: 200px;
        height: 200px;
        box-shadow: 0 0 15px #ddd;
      }
    }
    .submit-buttons {
      display: flex;
      justify-content: flex-end;
      align-items: center;
      width: 100%;
      height: 20%;
      padding: 0 1em;
      width: 100%;
      height: 5em;
      span.submit-img {
        width: 5em;
        height: 2.5em;
        line-height: 2.5;
        text-align: center;
        cursor: pointer;
        border-radius: 5px;
        border: 1px solid #ddd;
        transition: background 0.3s;
      }
    }
  }
  span.upload-img:hover, span.submit-img:hover {
    background: #eee;
  }
}
</style>

```
## 如果发现有Bug,希望能反馈给我
