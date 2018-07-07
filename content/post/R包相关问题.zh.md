+++
title = "R包安装和更新等相关问题"
date = 2018-07-03T13:55:47+08:00
draft = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ['R']
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

{{% toc %}}

R的生态环境近几年越来越好，但是包管理系统始终...反正我自己是遇到好多坑。又没有Maven那种，很难受。就在这里记录一下大坑们和解决办法吧...

## 更新相关

### R更新之后出现`Error: package * was installed by an R version with different internals; it needs to be reinstalled for use with this R version`
R大版本更新（3.4.3 -> 3.5）之后好多R包不能用了，报错信息如上。重装了一个，又会发现还有其他的包也有这种提示，子子孙孙无穷尽也...

解决方案：运行`update.packages(checkBuilt=TRUE, ask=FALSE)`将有必要更新的一股脑更新了去。

## 安装相关

### 使用`devtools::install_github()`报错`error setting certificate verify locations`

详细如下：

```pre
Installation failed: error setting certificate verify locations:
  CAfile: microsoft-r-cacert.pem
  CApath: none
```

解决方案：`install.packages(c("curl", "httr"))`，期间如果需要重启RSession会出提示。

### 使用`devtools::install_github()`报错BiocInstaller安装不上

详细如下：

```pre
Downloading GitHub repo bioinformatist/LncPipeReporter@master
from URL https://api.github.com/repos/bioinformatist/LncPipeReporter/zipball/master
Installing LncPipeReporter
'BiocInstaller' must be installed to install Bioconductor packages.
Would you like to install it? This will source <https://bioconductor.org/biocLite.R>.

1: Yes
2: No
```

这个时候玩家一般会天真的选1，然后就发现：

```pre
Installation failed: cannot open the connection to 'https://bioconductor.org/biocLite.R'
Warning message:
In file(filename, "r", encoding = encoding) :
  URL 'https://bioconductor.org/biocLite.R': status was 'Problem with the SSL CA cert (path? access rights?)'
```

先提供一个简单的解法，只要`source('http://bioconductor.org/biocLite.R')`之后重新运行`devtools::install_github()`安装包就无痛了。

注意到我只是**把https改成了http**，究竟影响了什么？

这就要提到一个历史遗留问题了：

devtools包历来都是用Linux自带的curl作为下载工具的，这点我有提过[issue](https://github.com/r-lib/devtools/issues/1641)，希望换成别的更高效的工具。但是从祖师爷Handley的回复看来，应该用curl也只是权宜之计，将来会有基于R的这样一个工具出来，那我就释怀了。

curl去访问下载https链接，是需要[证书](https://curl.haxx.se/docs/caextract.html)的！

我很奇怪，按说应该是有的呀，所以`curl-config --ca`了一下，发现在`/opt/anaconda/ssl/cacert.pem`。啊啊啊？

这个时候我想起来，服务器上装过Anaconda，之后为了方便大家使用conda等安装常用的生物信息分析工具，我就把`/opt/anaconda/bin`加入`PATH`了。然后其也带有一个低版本（真的是很旧）的curl，将操作系统本身的curl覆盖了。

{{% alert note %}}
Anaconda官方是非常不建议将`bin`目录加入`PATH`的，因为会有大量系统工具被覆盖（文件没有被替换，只是默认调用的都是Anaconda提供的版本）。Anaconda提供的这些binary文件也通常不会是最新的，因为它的哲学是“提供稳定可用的发行版”。但是这一“稳定”也许会和操作系统的使用相冲突。
{{% /alert %}}

意识到这个问题，我先短暂地将`/opt/anaconda/bin`从`PATH`中移除了。然后再去尝试安装，发现还是同样的报错。这个时候我就有一定程度的不适了。
我服务器上使用的R是MRO，MRO其实是提供了证书的（比如我的在`/usr/lib/R/lib/microsoft-r-cacert.pem`），我以为是MRO的bug，它调用curl的时候找不到这个证书，于是我天真地`Sys.setenv(CURL_CA_BUNDLE = "/usr/lib/R/lib/microsoft-r-cacert.pem")`指定了一下这个位置，结果发现还是不行。

这个时候我忽然想到，我的操作系统是Manjaro，更新太快导致curl和MRO内提供的证书版本不符。

看来还是用回Anaconda的旧版本curl+旧版本证书吧。

所以再把它写回`PATH`里面，然后`Sys.setenv(CURL_CA_BUNDLE = "/opt/anaconda/ssl/cacert.pem")`。终于能用了！

最后，如果是国内玩家，一直出现下述字样的话，还是改成http吧...

```pre
Installation failed: cannot open the connection to 'https://bioconductor.org/biocLite.R'
Warning message:
In file(filename, "r", encoding = encoding) :
  URL 'https://bioconductor.org/biocLite.R': status was 'Timeout was reached'
```