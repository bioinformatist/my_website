+++
date = "2017-07-05T09:37:43+08:00"
highlight = true
math = false
tags = ['network']
title = "How to use server that sits behind NAT"
summary = 'A post that teach you for bypassing corporate firewall to make your server usable worldwide'

[header]
  caption = ""
  image = ""

+++

```bash
ssh -NfR \*:666:127.0.0.1:22 root@138.197.194.11 -p 22 -b 0.0.0.0
```

`/etc/ssh/sshd_config`

```pre
GatewayPorts yes
```

`systemctl restart sshd.service`

https://linux.cn/article-5975-1.html