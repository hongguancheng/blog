---
title: 前端工程-自动上传 CDN
thumbnail: http://honggc.b0.upaiyun.com/blog/aoc7tslb1o8-lauren-mancke.jpg
---
最近有个朋友跟我说到每次上线好麻烦，项目每次 build 完之后都需要打开 cdn 的工具来上传静态的文件，然后复制 CDN 的 URL 进行在相应的位置进行替换。我突然想起了我一年前也碰到类似的问题，当时我还没有操作 CDN 的权限还要别人帮忙上传，上一次线十分的麻烦。于是当时做了个自动化上传 cdn 的工具，大大减少了重复性的工作。

当时用的是 grunt，所以写的是 [grunt 的插件](https://github.com/hongguancheng/grunt-enai-upyun)，用法好简单，在 grunt build 完之后，执行 grunt enai_upyun 就可以把生成好的目录中的所有文件都上传到 CDN 上，好方便有木有。

后来我们改用 webpack 作为构建工具，所以我用了类似的方法写了一个 [npm script 版](https://github.com/hongguancheng/npm-scripts-cdn)的上传工具，用法也差不多，后来我结合了 make 组合了下构建命令：

```txt
build:
	npm run build

upload-upyun: build 
	npm run enaiUpyun
	
release: 
	make upload-upyun
```

只需要 make release 一句命令就好了。懒的不行哈哈哈哈哈。

虽然我写的都是基于 upyun 的，但比如其他的服务比如七牛什么的都有相应的 SDK，可以根据自己的需求扩展一下，我把代码都放到了 Github 上了：

* [grunt-cdn-upyun](https://github.com/hongguancheng/grunt-enai-upyun)
* [npm-scripts-cdn](https://github.com/hongguancheng/npm-scripts-cdn)