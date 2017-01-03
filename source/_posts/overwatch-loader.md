---
title: 用 CSS3 做守望先锋的 loader
thumbnail: http://ww2.sinaimg.cn/large/81f090f2gw1f9wdmw1mb9j211e0hy7dl.jpg
<!-- thumbnail: http://ww3.sinaimg.cn/large/81f090f2gw1f9wflamsnzj212y0ksqe6.jpg -->
---
东西是前阵子写的现在拿出来说下，记得当时守望先锋刚出来没多久，因为电脑比较烂，加载地图速度一直好慢。。。等的时候就一直看着右下角的 loading 动画，看着看着就想着能不能用 CSS3 实现出来，后来为了研究这个动画进了好多好多次训练房间，最后弄出来还原度还是挺高的，效果如下：

{% codepen CCG EyGjqy 0 result 400 %}

<br /> 

### 中间 logo 部分的实现

中间的 logo 是由一个大圆环，两个平行四边形还有两个三角形组合而成的，平行四边形用 transform: skew 实现，三角形用 border 就可以了。

### 圆环动画效果

先说说圆环的实现，圆环有几种实现的方法，第一种是直接用 border

```css
.overwatch {
    border: 10px solid transparent;
    border-top: 10px solid #B6B8C0;
}
```

效果：

![圆环](http://honggc.b0.upaiyun.com/blog/%E5%9C%86%E7%8E%AF.jpg)

这样就能实现一个 1/4 园环，但是动画中的圆环大小不一所以明显不适合。我这边巧用了 clip 实现。

```css
.ring {
    width: 220px;
    height: 220px;
    border: 6px solid rgba(161, 164, 176, 0.5);
    border-radius: 50%;
    clip: rect(0px, 142px, 100px, 90px);
}
```

效果：

![圆环](http://honggc.b0.upaiyun.com/blog/%E5%9C%86%E7%8E%AF2.jpg)

这样就可以简单实现一些小的圆环了，但是用这种圆环来做动画效果非常的蛋疼，要动过 clip 的变化来使圆环大小变化，而且在圆环边缘位置的切线并不是垂直的，不够美观，所以我这里只用这种方法来实现一些小变化不大的圆环，而大的变化大的圆环我用了实现圆形进度条的方法来实现，具体 [看这里](https://www.xiabingbao.com/css/2015/07/27/css3-animation-circle.html)。

最后添加动画效果就可以实现最终效果了，出了这个我还用了类似的方法实现了[另一个守望先锋的 loading 动画](http://codepen.io/CCG/pen/KrANmJ)，还原度也是挺好的。