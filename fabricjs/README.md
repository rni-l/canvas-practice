# 学习使用 fabric.js

[学习记录，存在记录文件里](https://github.com/yiiouo/yiiouo.github.io/blob/master/recording/fabricjs%E7%9A%84%E5%AD%A6%E4%B9%A0%E4%BD%BF%E7%94%A8.md)

## 遇到的问题

### 放大图形时，总体在缩放

当我们 `canvas.add(new fabric.Rect())` ，添加的 rect（或者其他对象），使用 control 的缩放功能，我们可以看下 rect 实例的属性，width 和 height 是没有变化的，top 和 left 是有变化的。库里面是使用 scale 值进行缩放，所以无论是 stroke 还是 font-size 值等，都会被放大。

有时候需求只要你放大宽高而已，其他值不需要放大

```javascript
canvas.on('object:scaling', () => {
 const o = canvas.getActiveObject()
 if (!o.strokeWidthUnscaled && o.strokeWidth) {
   o.strokeWidthUnscaled = o.strokeWidth;
 }
 if (o.strokeWidthUnscaled) {
   o.strokeWidth = o.strokeWidthUnscaled / o.scaleX;
 }
})
```

`strokeWidthUnscaled` 相当于生成 rect 对象时，strokeWidth 的值，不变的。然后根据缩放的值，对 strokeWidth 进行修正。

但是这有个问题，在缩放时，strokeWidth 还是修正前的样子，缩放结束后，才是修正后的。我试过在其他的事件使用，但是无效~~

#### 另外一种解决方法

这种可能比较麻烦，但是可以真正解决这个问题。

```javascript
const rect = new fabric.Rect({
  id,
  name: 'rect',
  width: pointer.x - origX,
  height: pointer.y - origY,
  left: x,
  top: y,
  fill: 'rgba(255,0,0,0)'
})
const label = new fabric.Rect({
  id,
  name: 'label',
  width: pointer.x - origX,
  height: pointer.y - origY,
  left: x,
  top: y,
  fill: 'rgba(255,0,0,0)',
  hasBorders: false,
  hasControl: false,
  selectable: false,
  strokeWidth,
  stroke: '#838383'
})
```

在生成 rect 的同时，再生成多一个一模一样的 rect，声明为 label。这个 label 不能有 control 控件，不能被选中，有边框值。而 rect 什么值都没有，但是可以缩放移动。当 rect 被缩放移动时，修改 label 的 left top width height。

相当于 rect 是一个拖动缩放的模块，其他类似边框、字体，反正不想被缩放的对象，就单独生成一个新的对象，只修改它的 left top width height 值。

这时候大家可能会想到 Group 对象，但是 Group 对象被缩放，它的子级也会被缩放，这也是个问题~~

大家可以看下 test3.html 这个文件，试下~