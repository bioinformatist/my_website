+++
title = "如何优雅地在Linux操作系统下翻墙（客户端篇）"
date = 2018-05-29T13:59:17+08:00
draft = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['Linux', '计算机网络']
summary = ""

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
# Use `caption` to display an image caption.
#   Markdown linking is allowed, e.g. `caption = "[Image credit](http://example.org)"`.
# Set `preview` to `false` to disable the thumbnail in listings.
[header]
image = ""
caption = ""
preview = true

+++

本文主要介绍如何在Arch Linux下部署client端的SS，考虑到主要也是国人才有这种需求，就直接丢中文版blog了。

废话少说，`shadowsocks-libev`是C语言编写的轻量版的SS。先安装：
```shell
sudo pacman -S shadowsocks-libev
```

写好配置文件（具体写法就见[英文版教程]({{< ref "post/break-through-gfw.md" >}})吧，几乎一致的）：
```shell
sudo mkdir /etc/shadowsocks
# 我理你用nano还是vim还是个什么鬼，就编辑一下文本文件嘛
# 做人呢，最紧要是开心
sudo vim /etc/shadowsocks/config.json
```

使用`systemctl`管理ss的daemon进程（这个在介绍[Manjaro Linux的使用]({{< ref "post/working-on-linux.md" >}})的时候提过）：
```shell
# 配置文件的前缀名为config，因此此处为@config
sudo systemctl enable shadowsocks-libev@config
sudo systemctl daemon-reload
sudo systemctl start shadowsocks-libev@config
```

最后，使用`proxychains-ng`做socks5转发。

> 插嘴一句，本身就是齐胖胖他们医院的网络爆炸，所以只好搞这样一出来安装RStudio Server。没想到回到博士课题组修电脑还是用到了hhhhh

先用命令`sudo pacman -S proxychains-ng`安装，然后同样也要写配置，就很简单了，直接`sudo vim /etc/proxychains.conf`编辑，把原来`[ProxyList]`下面的内容去掉，加上`socks5 127.0.0.1 1080`。

接下来就是使用了。只要在需要使用代理的命令前面加上`proxychains4`就可以，比如`proxychains4 aurman -S rstudio-server-bin`这样。