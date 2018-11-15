---
title: 如何用纯 css 实现复杂图片
date: 1542177334000
---
一直以来我对一些漂亮的设计和图片都很感兴趣，也竟然会想着用 css 去把他们实现出来来提升一下对 css 的掌握程度，下面是我从一位设计小哥偷来的一张图片👻，今天讲的就是如何用纯 css 去实现它。

![psd](http://honggc.b0.upaiyun.com/blog/solor-system/psd.jpg)

#### 先从背景开始

背景我用了 radial-gradient 来实现背景的渐变效果。我用了 sass 的一些函数来方便取色。

```scss
.solar-system{
  // 一些代码
  background: radial-gradient(600px circle at 0% 50%, transparentize($sun-light, 0.9), transparent),
              radial-gradient(172px circle at 0% 50%, transparentize(#F2CEA5, .8), transparentize(#786755, .8) 171px, transparent 172px),
              radial-gradient(131px circle at 0% 50%, transparentize(#F7D2A9, .8), transparentize(#A99075, .8) 130px, transparent 131px),
              $background;
}
```
pug 和 sass 是用来做循环的好工具，在实现背景的星星的时候，首先用 pug 循环 20 个 element，sass 来循环 css 选择器的部分并且 sass 还能创建一些随机数来实现星星的随机分布。

```pug
  .stars
    - for (var i = 0; i < 20; i++)
      .star
```

```scss
.stars{
  @for $i from 1 through (20) {
    $starSize: random(4);
    $randomX: random($systemWidth);
    $randomY: random($systemHeight);
    .star:nth-of-type(#{$i}) {
      width: $starSize + px;
      height: $starSize + px;
      top: $randomY + px;
      left: $randomX + px;
      background-color: #C3E1FC;
      box-shadow: 0px 0px random(3) + 3 + px #C3E1FC;
      border-radius: 50%;
      opacity: (random(6) + 4) / 10;
    }
  }
}
```

接下来在加多几天行星公转的辅助线就完成了我们的背景

![背景]()

#### 行星的实现

以小行星带靠左的行星可以简单用一个标签实现，用 box-shadow 来实现阴影的部分。这里简单举一个🌰

```scss
.mercury{
  top: 50%;
  left: 172px;
  width: 24px;
  height: 24px;
  transform: translate(0, -50%);
  border-radius: 50%;
  background-color: $mercury-color;
  box-shadow: inset -4px 0px 0px darken($mercury-color, 20%),
              0px 0px 4px lighten($mercury-color, 10%);
}
```

小行星带靠右的行星跟左侧的主要区别在于球面上弧形的线条，这里还是用到了 box-shadow，用一子元素的多重阴影来实现

![tu1]()

![tu2]()

```scss
.jupiter{
  top: 50%;
  left: 440px;
  width: 60px;
  height: 60px;
  transform: translate(0, -50%);
  border-radius: 50%;
  background-color: $jupiter-color;
  overflow: hidden;
  box-shadow: 0px 0px 4px lighten($jupiter-color, 10%);
  .decoration{
    top: -50px;
    left: 50%;
    width: 130px;
    height: 50px;
    background: #fff;
    transform: translate(-50%, 0) rotate(-15deg);
    transform-origin: 65px 80px;
    border-radius: 50%;
    box-shadow: 0px 10px #BEB7AF,
                0px 14px $jupiter-color,
                0px 16px #B69C87,
                0px 20px $jupiter-color,
                0px 28px #B69C87,
                0px 32px $jupiter-color,
                0px 48px #B69C87,
                0px 50px $jupiter-color,
                0px 60px #BEB7AF;
  }
}
```

#### 小行星带
小行星带跟背景的星星差不多也是用 pug 循环创建出多个 element，用 sass 来循环写出每个 element 的 css。但是有一点不同的是小行星是在固定的轨道中随机的分布而不像背景星星全屏随机分布。这里同样是要借助 sass 的帮忙，已知小行星到太阳的距离，先随机出一个坐标 Y 然后通过勾股定理计算出相应的坐标 X 值。因为 sass 并没有 sqrt 方法，我们需要自己去实现。得到 Y 和 X 的值后还需要加上一个偏移值。

```scss
@function sqrt ($x) {
  @if $x < 0 {
      @warn "Argument for `sqrt()` must be a positive number.";
      @return null;
  }
  $ret: 1;
  @for $i from 1 through 24 {
      $ret: $ret - ($ret * $ret - $x) / (2 * $ret);
  }
  @return $ret;
}

.asteroid-belt{
  @for $i from 1 through (140) {
    $asteroidSize: random(2);
    $asteroidYPosition: random(300);
    $randomX: random(20) - 10;
    $randomY: random(20) - 10;
    .asteroid:nth-of-type(#{$i}) {
      width: $asteroidSize + px;
      height: $asteroidSize + px;
      top: 300 + $asteroidYPosition + $randomY + px;
      left: sqrt($asteroid-belt-radius * $asteroid-belt-radius - $asteroidYPosition * $asteroidYPosition) + $randomX + px;
      background-color: #fff;
      box-shadow: 0px 0px 2px #fff;
      border-radius: 50%;
      opacity: .5;
    }
  }
}
```

但是这时小行星只会在下发一侧分布，我们还需让小行星上下分布，通过 sass 的 random 加 条件判断就可实现。

```scss
.asteroid-belt{
  @for $i from 1 through (140) {
    $asteroidSize: random(2);
    $asteroidYPosition: random(300);
    $randomX: random(20) - 10;
    $randomY: random(20) - 10;
    .asteroid:nth-of-type(#{$i}) {
      width: $asteroidSize + px;
      height: $asteroidSize + px;
      @if random(2) == 1 {
        top: 300 + $asteroidYPosition + $randomY + px;
      } @else {
        top: 300 + $asteroidYPosition * -1 + $randomY + px;
      }
      left: sqrt($asteroid-belt-radius * $asteroid-belt-radius - $asteroidYPosition * $asteroidYPosition) + $randomX + px;
      background-color: #fff;
      box-shadow: 0px 0px 2px #fff;
      border-radius: 50%;
      opacity: .5;
    }
  }
}
```

图

这样我们就完成了我们的太阳系插画了，这里面多次用到 pug 和 sass 帮我们去做一些重复的工作，且介绍了一些 sass 实用的用法，希望能帮助到大家。