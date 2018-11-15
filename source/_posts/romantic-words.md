---
title: 手把手教你撸一个泡妞神奇
thumbnail: http://honggc.b0.upaiyun.com/blog/stars.jpg
date: 1484064000000
---

用 canvas 做的一个漫天星空，星星变成字，自己把自己感动的表白页面

其实东西是一年前写的，二话不说，先看[效果](http://honggc.b0.upaiyun.com/romantic-words/index.html)（点击屏幕可显示下一句）

![stars](http://honggc.b0.upaiyun.com/blog/starsss.gif)

当时我是在 codepan 上看到一个很漂亮的 [pan](http://codepen.io/iamfrontender/details/yNVPeX)，漫天星空，男孩独自看着，当时我就想如果可以把星星变成字就好了，于是就写了字的那一部分，背景还是用原来的，写完就用来表白了哈哈哈哈，效果怎么样嘛~~~反正就是追到了，哇哈哈哈哈，接下来说说是怎么做的吧。

### 具体实现

相信好多人一看就知道应该是用 canvas 做的了，具体做法就是在 canvas 画很多很多的点，然后根据你要显示的字准确排列，最后实现最后的效果。

#### 画字

首先我在画布上画了 1200 个点，用这些点来组成我们要显示的字，用不到的字就隐藏起来。在组成字之前我们需要知道每个点的具体的位置，这里的做法是首先在画布上用 ctx.fillText() 先画出我们要显示的字，然后用 ctx.getImageData() 得到画布上每个像素点的信息，在把这些像素点的信息转化为我们每个点的坐标，最后就能通过点来显示我们的字了，具体看代码：

```javascript
function draw () {
  ctx.clearRect(0, 0, CANVASWIDTH, CANVASHEIGHT)
  ctx.fillStyle = 'rgb(255, 255, 255)'
  ctx.textBaseline = 'middle'
  ctx.font = textSize + 'px \'Avenir\', \'Helvetica Neue\', \'Arial\', \'sans-serif\''
  ctx.fillText(text, (CANVASWIDTH - ctx.measureText(text).width) * 0.5, CANVASHEIGHT * 0.5)

  // 得到画布矩形的像素数据
  let imgData = ctx.getImageData(0, 0, CANVASWIDTH, CANVASHEIGHT)
  particleText(imgData)
}

function particleText (imgData) {
  // 点坐标获取
  var pxls = []
  for (var w = CANVASWIDTH; w > 0; w -= 3) {
    for (var h = 0; h < CANVASHEIGHT; h += 3) {
      var index = (w + h * (CANVASWIDTH)) * 4
      if (imgData.data[index] > 1) {
        pxls.push([w, h])
      }
    }
  }
  
  // 略
}
```

#### 点的运动

点在初始化的时候会被随机分布到画布的各个位置，在点的坐标确定之后，就会让点慢慢移动到目的地，具体的做法是在每一帧中根据点的上一帧的位置和点的目的地位置计算得出在该帧中点的坐标，让点慢慢的移动到目的地。

#### 星星闪烁效果

这个效果实现很简单，就是让星星不停的震动，具体就是让点的目的地坐标不停的进行小范围的偏移。具体请看代码：

```javascript
// 每次通过加上 Math.random() * 15 对目的地做偏移/
X = pxls[i - 1][0] - p.px + Math.random() * 15
Y = pxls[i - 1][1] - p.py + Math.random() * 15
```

代码都放到了 [github](https://github.com/hongguancheng/romantic-words) 上了，祝大家表白成功哈哈哈哈。

demo 演示：http://honggc.b0.upaiyun.com/romantic-words/index.html

github 地址：https://github.com/hongguancheng/romantic-words

背景地址：http://codepen.io/iamfrontender/details/yNVPeX