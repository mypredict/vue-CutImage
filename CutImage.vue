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
      theImage: '',
      theImg: '',
      fromDragImg: '',
      fromDrag: false,
      isCanvasShow: false,
      isHaveImg: false,
      isMoveInCanvas: false,
      isMousedown: false,
      isSlideMousedown: false,
      imgWH: 0,
      mouseX: 0,
      mouseY: 0,
      imgMoveLeft: 0,
      imgMoveTop: 0,
      imgMoveX: 0,
      imgMoveY: 0,
      slideFirstX: 0,
      slideFinalX: 0,
      firstImgWidth: 0,
      firstImgHeight: 0
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
      this.imgBoard.removeEventListener('drop', this.dropEvent)
      window.removeEventListener('mousedown', this.mousedownEvent)
      window.removeEventListener('click', this.clickEvent)
      window.removeEventListener('mousemove', this.mousemoveEvent)
      window.removeEventListener('mouseup', this.mouseupEvent)
      window.removeEventListener('mousewheel', this.mousewheelEvent)
      window.removeEventListener('DOMMouseScroll', this.DOMMouseScrollEvent)
    })
  },
  destroyed () {
    this.imgBoard.removeEventListener('drop', this.dropEvent)
    window.removeEventListener('mousedown', this.mousedownEvent)
    window.removeEventListener('click', this.clickEvent)
    window.removeEventListener('mousemove', this.mousemoveEvent)
    window.removeEventListener('mouseup', this.mouseupEvent)
    window.removeEventListener('mousewheel', this.mousewheelEvent)
    window.removeEventListener('DOMMouseScroll', this.DOMMouseScrollEvent)
  },
  methods: {
    focusInput () {
      document.getElementsByClassName('shade-floor')[0].style.display = 'block'
      document.getElementById('taggle-img').click()
    },
    allowDrop (e) {
      e.preventDefault()
    },
    uploadSuccess () {
      if (!this.isCanvasShow) {
        this.isCanvasShow = true
        this.theCanvas.width = this.canvasWidth
        this.theCanvas.height = this.canvasHeight
      }
      this.slideFirstX = this.slideLineWidth / 2
      this.slideBlock.style.left = this.slideFirstX + 'px'
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
          this.isHaveImg = true
          document.getElementById('taggle-img').value = null
          this.imgWH = this.theImg.width / this.theImg.height
          if (this.imgWH >= this.canvasWH) {
            this.theImg.width = this.canvasWidth
            this.theImg.height = this.theImg.width / this.imgWH
            this.imgMoveX = this.imgMoveLeft = 0
            this.imgMoveY = this.imgMoveTop = (this.canvasHeight - this.theImg.height) / 2
          } else {
            this.theImg.height = this.canvasHeight
            this.theImg.width = this.theImg.height * this.imgWH
            this.imgMoveX = this.imgMoveLeft = (this.canvasWidth - this.theImg.width) / 2
            this.imgMoveY = this.imgMoveTop = 0
          }
          this.firstImgWidth = this.theImg.width
          this.firstImgHeight = this.theImg.height
          this.redrawImage()
        }
      }
    },
    listenMousewheel (e) {
      if (this.isHaveImg) {
        this.isMoveInCanvas = true
      }
    },
    removeMousewheel () {
      if (this.isHaveImg) {
        this.isMoveInCanvas = false
      }
    },
    redrawImage () {
      this.canvasCtx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.canvasCtx.drawImage(this.theImg, this.imgMoveLeft, this.imgMoveTop, this.theImg.width, this.theImg.height)
      let imgClipData = this.canvasCtx.getImageData(this.clipLightX, this.clipLightY, 400, 400)
      this.rightCanvasCtx.clearRect(0, 0, 200, 200)
      this.rightCanvasCtx.putImageData(imgClipData, 0, 0)
      this.canvasCtx.save()
      this.canvasCtx.fillStyle = 'rgba(0, 0, 0, 0.4)'
      this.canvasCtx.fillRect(0, 0, this.canvasWidth, this.canvasHeight)
      this.canvasCtx.rect(this.clipLightX, this.clipLightY, 400, 400)
      this.canvasCtx.fillStyle = 'rgba(255, 255, 255, 10)'
      this.canvasCtx.fill()
      this.canvasCtx.restore()
      this.canvasCtx.save()
      this.canvasCtx.beginPath()
      this.canvasCtx.rect(this.clipLightX, this.clipLightY, 400, 400)
      this.canvasCtx.closePath()
      this.canvasCtx.clip()
      this.canvasCtx.drawImage(this.theImg, this.imgMoveLeft, this.imgMoveTop, this.theImg.width, this.theImg.height)
      this.canvasCtx.restore()
    },
    dropEvent (e) {
      e.preventDefault()
      this.fromDrag = true
      this.fromDragImg = e.dataTransfer.files[0]
      if (this.fromDragImg.type.indexOf('image/') === 0) {
        this.uploadSuccess()
      } else {
        alert('请上传正确的图片!!!')
      }
    },
    mousedownEvent (e) {
      if (e.target === this.slideBlock && this.isHaveImg) {
        this.isSlideMousedown = true
        this.mouseX = e.clientX
      }
      if (e.target === this.theCanvas && this.isHaveImg) {
        this.isMousedown = true
        this.mouseX = e.clientX
        this.mouseY = e.clientY
      }
    },
    clickEvent (e) {
      if (e.target === this.slideLine && this.isHaveImg) {
        let useMinLeft = this.theImg.width
        let useMinTop = this.theImg.height
        let theDistance = e.offsetX - this.slideFirstX
        this.slideFirstX = e.offsetX
        this.slideBlock.style.left = e.offsetX + 'px'
        this.theImg.width = this.firstImgWidth += theDistance
        this.theImg.height = this.theImg.width / this.imgWH
        if (Math.min(this.theImg.width, this.theImg.height) > 400) {
          this.imgMoveLeft = this.imgMoveX -= theDistance / 2
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
    },
    mousemoveEvent (e) {
      if (this.isMousedown) {
        this.imgMoveLeft = this.imgMoveX + (e.clientX - this.mouseX) * 2
        this.imgMoveTop = this.imgMoveY + (e.clientY - this.mouseY) * 2
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
      if (this.isSlideMousedown) {
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
          this.imgMoveLeft = this.imgMoveX - slideMoveX / 2
          this.imgMoveTop = this.imgMoveY - slideMoveX / 2 / this.imgWH
        }
        this.redrawImage()
      }
    },
    mouseupEvent (e) {
      if (this.isHaveImg) {
        this.isMousedown = false
        this.imgMoveX = this.imgMoveLeft
        this.imgMoveY = this.imgMoveTop
        if (this.isSlideMousedown) {
          this.isSlideMousedown = false
          this.slideFirstX = this.slideFinalX
          this.firstImgWidth = this.theImg.width
          this.firstImgHeight = this.theImg.height
        }
      }
    },
    mousewheelEvent (e) {
      if (this.isMoveInCanvas) {
        e = e || window.event
        e.preventDefault()
        this.theImg.width += e.wheelDelta / 10
        this.theImg.height = this.theImg.width / this.imgWH
        if (Math.min(this.theImg.width, this.theImg.height) > 400 || e.wheelDelta > 0) {
          this.imgMoveX = this.imgMoveLeft -= e.wheelDelta / 20
          this.imgMoveY = this.imgMoveTop -= e.wheelDelta / 20 / this.imgWH
          this.firstImgWidth = this.theImg.width
        } else {
          this.theImg.width -= e.wheelDelta / 10
          this.theImg.height = this.theImg.width / this.imgWH
          this.firstImgWidth = this.theImg.width
        }
        this.redrawImage()
      }
    },
    DOMMouseScrollEvent (e) {
      if (this.isMoveInCanvas) {
        e = e || window.event
        e.preventDefault()
        this.theImg.width -= e.detail * 4
        this.theImg.height = this.theImg.width / this.imgWH
        if (Math.min(this.theImg.width, this.theImg.height) >= 400 || e.detail < 0) {
          this.imgMoveX = this.imgMoveLeft += e.detail * 2
          this.imgMoveY = this.imgMoveTop += e.detail * 2 / this.imgWH
          this.firstImgWidth = this.theImg.width
        } else {
          this.theImg.width += e.detail * 4
          this.theImg.height = this.theImg.width / this.imgWH
          this.firstImgWidth = this.theImg.width
        }
        this.redrawImage()
      }
    },
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
  user-select: none;
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
