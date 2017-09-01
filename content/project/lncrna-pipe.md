+++
date = "2017-08-11T11:23:37+08:00"
external_link = ""
highlight = true
image_preview = ""
math = false
summary = ""
tags = ['software']
title = "lncRNA pipe"

[header]
  caption = ""
  image = ""

+++

```bash
sudo pacman -S docker docker-machine
docker version
```

Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

```bash
sudo systemctl enable docker.service
sudo vim /etc/systemd/system/multi-user.target.wants/docker.service
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30/images/json: dial unix /var/run/docker.sock: connect: permission denied

```bash
sudo usermod -aG docker $(whoami)
```
```bash
docker pull ubuntu
docker run -t -i ubuntu /bin/bash
```

Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

```bash
sudo sysctl -w net.ipv4.tcp_mtu_probing=1

# To make this permanently, add the following line to /etc/sysctl.conf:
net.ipv4.tcp_mtu_probing=1
```

```bash
until docker build -t bioinformatist/lnc-pipe:add-kallisto . ; do docker build -t bioinformatist/lnc-pipe:add-kallisto . ; done
```