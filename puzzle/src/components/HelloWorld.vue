<template>
  <div class='m-puzzle'>
    <div class='m-puzzle_wrap'>
      <div class='m-puzzle_content'>
        <canvas class='canvas'></canvas>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        oContent: {},
        oCanvas: {},
        ctx: {},
        opts: {
          width: 0,
          height: 0,
          offsetTop: 0,
          offsetLeft: 0,
          canMoveDis: 20,
          speed: 10
        },
        rectArea: 0,
        lineCoordinates: [],
        rectCoordinates: [],
        rects: [], // 拼图对象
        cacheData: {
          f_x: 0,
          f_y: 0
        },
        ifCanTouchMove: false,
        ifCanTouchStart: true,
        clickRectIndex: 0,
        moveDirection: 'none',
        vacancy: 0,
        endTheReturnCoordinates: {},
        selectedObj: {},
        isStopAnimation: true,
        wrapTouchMoveHandle: () => {},
        wrapTouchEndHandle: () => {}
      }
    },

    mounted() {
      this.initGetDomValue()
      this.computingContainerOffset()
      this.computingCoordinate()
      this.initRects()
      this.draw()
      this.oCanvas.addEventListener('touchstart', this.touchStart.bind(this), false)
    },

    methods: {
      initGetDomValue() {
        this.oCanvas = document.querySelector('.m-puzzle_content canvas')
        this.ctx = this.oCanvas.getContext('2d')
        this.oContent = document.querySelector('.m-puzzle_content')
        const width = this.oContent.offsetWidth
        const height = this.oContent.offsetHeight
        this.oCanvas.width = width
        this.oCanvas.height = height
        this.opts.width = width
        this.opts.height = height
      },

      computingContainerOffset() {
        const oPage = document.querySelector('.m-puzzle')
        const { width, height } = this.opts
        const bw = oPage.offsetWidth
        const bh = oPage.offsetHeight
        this.opts.offsetLeft = (bw - width) / 2
        this.opts.offsetTop = (bh - height) / 2
      },

      computingCoordinate() {
        // 根据矩形宽高，平分三份，获取对应的坐标
        const { width, height } = this.opts
        const levelAPart = width / 3
        const levelTwoPark = levelAPart * 2
        const columnAPark = height / 3
        const columnTwoPark = columnAPark * 2
        this.opts.sideLength = levelAPart
        this.rectArea = levelAPart * columnAPark
        this.lineCoordinates = [
          { x: levelAPart, y: 0, x1: levelAPart, y1: height },
          { x: levelTwoPark, y: 0, x1: levelTwoPark, y1: height },
          { x: 0, y: columnAPark, x1: width, y1: columnAPark },
          { x: 0, y: columnTwoPark, x1: width, y1: columnTwoPark }
        ]
        const a = []
        // 获取 9 个正方形，16 个点
        for (let i = 0; i < 16; i++) {
          const levelIndex = i % 4
          const _i = i
          const columnIndex = _i > 3 && _i < 8 ? 1 : (
            _i > 7 && _i < 12 ? 2 : (
              _i > 11 ? 3 : 0
            )
          )
          a.push({
            x: levelIndex * levelAPart,
            y: columnIndex * columnAPark
          })
        }
        for (let i = 0; i < 9; i++) {
          const _i = i / 3 < 1 ? i : (i + Math.floor(i / 3))
          const arr = []
          arr.push(a[_i])
          arr.push(a[_i + 1])
          arr.push(a[_i + 4])
          arr.push(a[_i + 5])
          this.rectCoordinates.push(arr)
        }
      },

      getTheDistanceBetweenTwoPoints(touchPoint, rectPoint) {
        return Math.sqrt(Math.pow(touchPoint.x - rectPoint.x, 2) + Math.pow(touchPoint.y - rectPoint.y, 2))
      },

      getTriangleArea(a, b, c) {
        const p = (a + b + c) / 2
        return Math.floor(Math.sqrt(p * (p - a) * (p - b) * (p - c)))
      },

      initRects() {
        const { sideLength } = this.opts
        this.rectCoordinates.forEach((v, i) => {
          if (i === 8) return false
          const { x, y } = v[0]
          this.rects.push({
            x,
            y,
            cachex: x,
            cachey: y,
            width: sideLength,
            height: sideLength,
            value: i,
            index: i
          })
        })
        this.vacancy = 8
      },

      drawRects() {
        this.rects.forEach((v) => {
          this.ctx.save()
          this.ctx.strokeStyle = 'red'
          this.ctx.strokeRect(v.x, v.y, v.width, v.height)
          this.ctx.fontSize = '16px'
          this.ctx.strokeText(v.value, Math.abs((v.width + v.x) - (v.width / 2)), Math.abs((v.height + v.y) - (v.height / 2)))
          this.ctx.restore()
        })
      },

      drawPoint(x, y) {
        this.ctx.beginPath()
        this.ctx.arc(x, y, 5, 0, Math.PI * 180 / 2, true)
        this.ctx.stroke()
        this.ctx.closePath()
      },

      drawPath(r, p) {
        this.ctx.beginPath()
        this.ctx.moveTo(r.x, r.y)
        this.ctx.lineTo(p.x, p.y)
        this.ctx.stroke()
        this.ctx.closePath()
      },

      computingPointIfInRect(x, y) {
        const point = { x, y }
        const rectArea = this.rectArea
        const line = this.opts.sideLength
        let clickRectIndex = 0
        this.rectCoordinates.some((rect, rectIndex) => {
          let area = 0
          const lineArr = []
          rect.forEach((line, lineIndex) => {
            lineArr.push(this.getTheDistanceBetweenTwoPoints(point, line))
            this.drawPath(line, point)
          })
          // 计算 4 个三角形的面积
          area += this.getTriangleArea(lineArr[0], lineArr[1], line)
          area += this.getTriangleArea(lineArr[1], lineArr[3], line)
          area += this.getTriangleArea(lineArr[2], lineArr[3], line)
          area += this.getTriangleArea(lineArr[2], lineArr[0], line)
          // 判断三角形面积和 是否等于矩形面积
          clickRectIndex = rectIndex
          return rectArea >= area
        })
        return clickRectIndex
      },

      computingRectMoveDirction(rectIndex) {
        let direction = ''
        const vacancy = this.vacancy
        if (rectIndex - 3 === vacancy) {
          direction = 'top'
        } else if (rectIndex + 3 === vacancy) {
          direction = 'bottom'
        } else if (rectIndex - 1 === vacancy) {
          direction = 'left'
        } else if (rectIndex + 1 === vacancy) {
          direction = 'right'
        } else {
          direction = 'none'
        }
        return direction
      },

      createLine() {
        this.ctx.lineWidth = 1
        this.lineCoordinates.forEach(v => {
          this.ctx.beginPath()
          this.ctx.moveTo(v.x, v.y)
          this.ctx.lineTo(v.x1, v.y1)
          this.ctx.stroke()
          this.ctx.closePath()
        })
      },

      getRectAccordingClickIndex() {
        return this.rects.filter(v => {
          return v.index === this.clickRectIndex
        })[0]
      },

      setRectMove(type, value) {
        const obj = this.selectedObj
        obj[type] = obj['cache' + type] + value
      },

      computingTheEndAnimation() {
        const direction = this.moveDirection
        const { speed } = this.opts
        const speedDirection = direction === 'top' || direction === 'left' ? -1 : 1
        const obj = this.selectedObj
        const { x, y } = this.endTheReturnCoordinates
        if (direction === 'left' || direction === 'right') {
          obj.x += speedDirection * speed
          if (direction === 'left') {
            if (obj.x <= x) {
              obj.x = x
              this.isStopAnimation = true
            }
          } else {
            if (obj.x >= x) {
              obj.x = x
              this.isStopAnimation = true
            }
          }
        } else {
          obj.y += speedDirection * speed
          if (direction === 'top') {
            if (obj.y <= y) {
              obj.y = y
              this.isStopAnimation = true
            }
          } else {
            if (obj.y >= y) {
              obj.y = y
              this.isStopAnimation = true
            }
          }
        }
      },

      touchEndAnimation() {
        if (this.isStopAnimation) {
          const obj = this.selectedObj
          obj.cachex = obj.x
          obj.cachey = obj.y
          this.ifCanTouchStart = true
          return false
        }
        this.computingTheEndAnimation()
        this.draw()
        requestAnimationFrame(this.touchEndAnimation.bind(this))
      },

      touchStart(e) {
        e.preventDefault()
        if (!this.ifCanTouchStart) return false
        const { clientX, clientY } = e.targetTouches[0]
        const { offsetLeft, offsetTop } = this.opts
        this.ifCanTouchMove = true
        const x = clientX - offsetLeft
        const y = clientY - offsetTop
        this.cacheData.f_x = x
        this.cacheData.f_y = y
        this.draw()
        this.clickRectIndex = this.computingPointIfInRect(x, y)
        // 计算该拼图，可以往哪个方向移动
        this.moveDirection = this.computingRectMoveDirction(this.clickRectIndex)
        // this.drawPoint(x, y)
        this.wrapTouchMoveHandle = this.touchMoveHandle.bind(this)
        this.wrapTouchEndHandle = this.touchEndHandle.bind(this)
        this.oCanvas.addEventListener('touchmove', this.wrapTouchMoveHandle, false)
        this.oCanvas.addEventListener('touchend', this.wrapTouchEndHandle, false)
        this.selectedObj = this.getRectAccordingClickIndex()
      },

      touchMoveHandle(e) {
        e.preventDefault()
        if (!this.ifCanTouchMove) return false
        const { offsetLeft, offsetTop } = this.opts
        const { clientX, clientY } = e.targetTouches[0]
        const m_x = clientX - offsetLeft
        const m_y = clientY - offsetTop
        const { f_x, f_y } = this.cacheData
        const direction = this.moveDirection
        console.log(direction)
        if ((direction === 'left' && f_x > m_x) || (direction === 'right' && f_x <= m_x)) {
          this.setRectMove('x', m_x - f_x)
        } else if ((direction === 'top' && f_y > m_y) || (direction === 'bottom' && f_y <= m_y)) {
          this.setRectMove('y', m_y - f_y)
        } else {
          return false
        }
        this.draw()
      },

      touchEndHandle() {
        this.ifCanTouchMove = false
        this.oCanvas.removeEventListener('touchmove', this.wrapTouchMoveHandle, false)
        this.oCanvas.removeEventListener('touchend', this.wrapTouchEndHandle, false)
        // 判断移动的位置是否超过指定值
        const { x, cachex, y, cachey, index } = this.selectedObj
        const { canMoveDis } = this.opts
        let endTheReturnIndex = 0
        if (Math.abs(x - cachex) >= canMoveDis || Math.abs(y - cachey) >= canMoveDis) {
          // move
          this.selectedObj.index = this.vacancy
          endTheReturnIndex = this.vacancy
          this.vacancy = index
        } else {
          // back
          endTheReturnIndex = index
          const direction = this.moveDirection
          if (direction === 'top') {
            this.moveDirection = 'bottom'
          } else if (direction === 'bottom') {
            this.moveDirection = 'top'
          } else if (direction === 'left') {
            this.moveDirection = 'right'
          } else if (direction === 'right') {
            this.moveDirection = 'left'
          }
        }
        // 计算要返回的坐标
        this.endTheReturnCoordinates = this.rectCoordinates[endTheReturnIndex][0]
        this.isStopAnimation = false
        this.ifCanTouchStart = false
        this.touchEndAnimation()
      },

      draw() {
        const { width, height } = this.opts
        this.ctx.clearRect(0, 0, width, height)
        this.createLine()
        this.drawRects()
      }
    }
  }
</script>

<style lang='less'>
  .m-puzzle {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh;
    &_wrap {
      width: 90%;
      border: solid 1px #000; /* no */
    }
    &_content {
      padding-top: 100%;
      width: 100%;
      position: relative;
      .canvas {
        position: absolute;
        top: 0;
        left: 0;
      }
    }
  }
</style>
