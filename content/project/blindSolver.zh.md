+++
title = "BlindSolver开发记录"
date = 2018-07-17T17:40:27+08:00
draft = false

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ['TypeScript', 'GUI']

# Project summary to display on homepage.
summary = ""

# Optional image to display on homepage.
image_preview = ""

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Does the project detail page use source code highlighting?
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++

如同以前开发[Gao's SB](https://github.com/bioinformatist/Gao-s-SB)一样，来到新课题组之后，发现还是有一些蛋疼的需求。我在生信狗里面算是编程能力还可以的，但是依然不想遇到问题就去摸一个一锤子买卖的小脚本出来。爪哥的[csvtk](https://github.com/shenwei356/csvtk)和[seqkit](https://github.com/shenwei356/seqkit)自然是优秀了，但是人懒到一定程度，觉得在命令行内用[rush](https://github.com/shenwei356/rush)配合这些神器搞风搞雨也是很难受的。所以我需要一个优秀的（开发快捷、逻辑优美以及跨平台方便打包）平台来再造一个GUI程序在将来方便我搞事情，所以我选择了符合以上哲学的TypeScript。当然，只要爪哥一天还在维护这些牛X工具，我都还是会在程序里调用这些轮子，毕竟短小精悍（误）又好用。

名字暂定为BlindSolver。开工！

> 如果你不幸在将来用到了我的轮子，并且觉得好用的话，那欢迎你加入开发:smile:

我是会写JS不错，但是这个所谓的“超集”真是把我吓到了。也是从头摸起的，先安装Node.js，然后`npm install -g typescript`，这样就好了。

对这个东西可能期望高一点，暂时不希望方向被带歪。所以选择不添加许可证了还是，自己写。

以前是用Pycharm做PyQt开发的，但是现在看了很多人的经验之后，觉得还是Eric比较好，python原生的IDE，由于是基于PyQt实现的界面，那么必然也是偏爱一些，那么用起来肯定舒服。

## 环境配置篇

Anaconda最新版，Eric最新版。安装的时候，提示缺少QSci，就自动安装好了。这里马上会出现一个问题，Anaconda已经自带了PyQt了，虽然就不是最新的版本（Anaconda都是很负责的啦，考虑兼容性的）。这样Eric环境配置的时候居然会自动安装新版本的PyQt。

以及有这样一个错误：

```pre
distributed 1.21.8 requires msgpack, which is not installed.
```

使用`conda install -c anaconda msgpack-python`补上缺少的库。

```pre
  The scripts pylupdate5.exe, pyrcc5.exe and pyuic5.exe are installed in 'C:\Users\sun_y\AppData\Roaming\Python\Python36\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
```

