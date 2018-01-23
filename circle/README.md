# 画出圆弧效果

> 用 canvas 画出圆弧的动画效果

## 绘制圆弧

使用 ctx.arc 就可以画出你想要的度数的圆弧

```
// 接受三个参数，x、y 位置，半径，开始弧度和结束度数，是否顺时针
ctx.arc(arcW, arcH, outRadius, startAngle, endAngle, true)

// 源码
ctx.save()
ctx.scale(smallScale, smallScale)
ctx.beginPath()
ctx.arc(arcW, arcH, outRadius, startAngle, endAngle, true)
ctx.arc(arcW, arcH, insideRadius, endAngle, startAngle, false)
ctx.closePath()
ctx.stroke()
ctx.fillStyle = 'red'
ctx.fill()
ctx.restore()

```

beginPath、closePath 这两个的作用要明白。beginPath 是声明一个开始的路径，closePath 是把路径合并起来。

上面的代码，先 beginPath 再画圆，然后再次画圆弧，这样的话，两个圆弧之间会有一边被合并，然后再 closePath，关闭路径，两个圆弧就回完整合并。

如果只想弄出两条圆弧的效果

```
ctx.beginPath()
ctx.arc(arcW, arcH, outRadius, startAngle, endAngle, true)
ctx.beginPath()
ctx.arc(arcW, arcH, insideRadius, endAngle, startAngle, false)
```

再次使用 beginPath 就好了

## 减少锯齿方法

如果使用 ctx.arc 画出一个圆弧，其实是有锯齿的，这里使用一种放大缩小的做法，令锯齿尽量减少。先把要画的物体的值，乘以 5 倍，然后在画之前，使用 `ctx.scale` 进行缩小，达到减少锯齿的效果。

## 周期计算

每当超过 360 自动从 0 开始，这样做的话感觉会很复杂，所以直接在 360 继续累积加上去。比如 170 - 10，这个圆弧，就是 170 - 370 这个幅度，相差 200 度。

## 在圆弧上，平均划分 (TODO)

在圆弧上，平均分成 5 分，出现线条进行分割。现在想到的是获取每条线的坐标，进行绘制。但是怎么获取到坐标呢。。。。嗯还是个问题~~
