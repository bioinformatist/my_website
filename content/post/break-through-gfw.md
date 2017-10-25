+++
tags = ['network']
categories = []
highlight = true
date = "2017-02-15T22:48:04+08:00"
title = "Break through GFW blockade with shadowsocks built on Digital Ocean"
math = false
image = ""
summary = "Guide to break through Chinese government's GFW"

+++

1. If you're a student, try apply for a [student pack](https://education.github.com/pack/) on *GitHub*.
2. Use code of *Digital Ocean* in student pack to register a **new account** and buy a droplet at *San Fransico* (both node 1 & 2 is OK, maybe 2 is better).
3. Install *Fedora 25 (64 bit)* on your VPS then do as below:

Install shadowsocks:

```shell
sudo dnf install m2crypto python-setuptools
easy_install pip
pip install shadowsocks
```

Shadowsocks use a json file as configuration. For all users, this file can be placed in `/etc`.
Use `vi  /etc/shadowsocks.json` to edit the file and fill in contents below:

```json
{
    "server":"0.0.0.0",
    "server_port":"port",
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"passwd",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
}
```

To keep shadowsocks running in background after you cut off SSH connection, you may use `supervisor`.

```shell
sudo dnf install supervisor
echo_supervisord_conf > /etc/supervisord.conf
```

Execute `vi /etc/supervisord.conf` to configure it:

```pre
[program:shadowsocks]
command=ssserver -c /etc/shadowsocks.json
autostart=true
autorestart=true
user=nobody
```

Finally, `supervisord` or `supervisord -c /etc/supervisord.conf` to start shadowsocks server and visit [the project's homepage](https://github.com/shadowsocks) on GitHub for a client.
For Windows user, try https://github.com/shadowsocks/shadowsocks-windows/releases.
For Linux user, [shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5/releases) is a good choice.
There're also [shadowsocks for Android](https://github.com/shadowsocks/shadowsocks-android/releases) and [shadowsocks for iOS](https://github.com/shadowsocks/shadowsocks-iOS/releases).

## Appendix

`supervisorctl status` for checking all processes' status controlled by *supervisor*.

`supervisorctl stop ssserver` for kill a process.

`supervisorctl start ssserver` for start a process.

`supervisorctl shutdown` for killing all processes controlled by *supervisor*.

## Known issue

Sometimes shadowsocks works fine in *Windows*,
but your browser can not visit pages in *Arch Linux*, 
it is because that too frequently updating `libQtShadowsocks` brings conflict.

To [solve this problem](https://github.com/shadowsocks/shadowsocks-qt5/issues/550),
you can downgrade `libQtShadowsocks` or change the encryption method from `AES-256-CFB` to `AES-256-CTR`.