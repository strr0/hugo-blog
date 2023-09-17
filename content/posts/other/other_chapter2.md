---
title: "Lightweight proxy on Linux"
date: 2023-09-15T17:00:00+08:00
draft: false
---

### Usage

Install shadowocks

```
yay -S shadowsocks-rust
```

Start local client with configuration file

```
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json
```

#### Socks5 Local client

```
# Pass all parameters via command line
sslocal -b "127.0.0.1:1080" -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "v2ray-plugin" --plugin-opts "server;tls;host=github.com"

# Pass server with SIP002 URL
sslocal -b "127.0.0.1:1080" --server-url "ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ@127.0.0.1:8388/?plugin=v2ray-plugin%3Bserver%3Btls%3Bhost%3Dgithub.com"
```

#### HTTP Local client

```
sslocal -b "127.0.0.1:3128" --protocol http -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty"
```

All parameters are the same as Socks5 client, except `--protocol http`.

### **Docker**

This project provided Docker images for the `linux/i386` and `linux/amd64` and `linux/arm64/v8` architectures.

> ⚠️ **Docker containers do not have access to IPv6 by default**: Make sure to disable IPv6 Route in the client or [enable IPv6 access to docker containers](https://docs.docker.com/config/daemon/ipv6/#use-ipv6-for-the-default-bridge-network).

#### Pull from GitHub Container Registry

Docker will pull the image of the appropriate architecture from our [GitHub Packages](https://github.com/orgs/shadowsocks/packages?repo_name=shadowsocks-rust).

```
docker pull ghcr.io/shadowsocks/sslocal-rust:latest
docker pull ghcr.io/shadowsocks/ssserver-rust:latest
```


#### Build on the local machine（Optional）

If you want to build the Docker image yourself, you need to use the [BuildX](https://docs.docker.com/buildx/working-with-buildx/).

```
docker buildx build -t shadowsocks/ssserver-rust:latest -t shadowsocks/ssserver-rust:v1.15.2 --target ssserver .
docker buildx build -t shadowsocks/sslocal-rust:latest -t shadowsocks/sslocal-rust:v1.15.2 --target sslocal .
```


#### Run the container

You need to mount the configuration file into the container and create an external port map for the container to connect to it.

```
docker run --name sslocal-rust \
  --restart always \
  -p 1080:1080/tcp \
  -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
  -dit ghcr.io/shadowsocks/sslocal-rust:latest

docker run --name ssserver-rust \
  --restart always \
  -p 8388:8388/tcp \
  -p 8388:8388/udp \
  -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
  -dit ghcr.io/shadowsocks/ssserver-rust:latest
```

Create a ShadowSocks' configuration file. Example

```
{
    "server": "my_server_ip",
    "server_port": 8388,
    "password": "rwQc8qPXVsRpGx3uW+Y3Lj4Y42yF9Bs0xg1pmx8/+bo=",
    "method": "aes-256-gcm",
    // ONLY FOR `sslocal`
    // Delete these lines if you are running `ssserver` or `ssmanager`
    "local_address": "127.0.0.1",
    "local_port": 1080
}
```

### Browser Setting

Install privoxy

```
pacman -S privoxy
```

add privoxy conifg

```
/etc/privoxy/config
forward-socks5 / 127.0.0.1:1080 .
listen-address  127.0.0.1:8118
```

start privoxy and browser

```
systemctl start privoxy
chromium --proxy-server="http://127.0.0.1:8118"
```