+++
title = "How to use server that sits behind NAT"

date = 2017-07-05T09:37:43
draft = true

tags = ["network"]
+++

```bash
ssh -NfR \*:666:127.0.0.1:22 root@138.197.194.11 -p 22 -b 0.0.0.0
```

`/etc/ssh/sshd_config`

```pre
GatewayPorts yes
```

`systemctl restart sshd.service`

http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/