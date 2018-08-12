+++
title = "Teamviewer命令行"
date = 2018-08-12T13:20:14+08:00
draft = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['Linux']
categories = []

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

新安装了服务器的操作系统，然后不想爬上爬下接个显示器（实验室的服务器放在月亮之上），要怎么样才能完全从命令行配置好teamviewer？

先安装：`aurman -S teamviewer`。

然后，参照[之前的日记]({{< ref "post/working-on-linux.md" >}})中记载的做法，将teamviewer的daemon设置好。

为了显示teamviewer的ID，先得有个密码。否则出于安全考虑，是不会显示ID的：

```shell
sudo teamviewer --passwd yourpasswd
```

这样再`sudo teamviewer --info`就能看到ID了。

拿到了ID和密码，就能直接连接啦~

好了，接下来，我又吃精地发现一个问题，连接持续了几秒钟而已，挂了；再连接，连不上...我滚去一台局域网内的电脑在终端内尝试`teamviewer`，发现：

```pre
Init...
CheckCPU: SSE2 support: yes
Checking setup...
Launching TeamViewer ...
Launching TeamViewer GUI ...
Aborted (core dumped)
```

折腾了很久，终于找到了[解决办法](https://forum.getpimp.org/topic/795/teamviewer-13-on-pimp-v2-4-1-all/2)：

`sudo nano /opt/teamviewer/config/global.conf`加上以下两行配置：

```pre
[int32] EulaAccepted = 1
[int32] EulaAcceptedRevision = 6
```

再重启daemon服务，可以了！