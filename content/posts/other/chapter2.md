---
title: "Proxy on linux"
date: 2023-09-15T17:00:00+08:00
categories: ["other"]
draft: false
---

## Overview
proxy

## 1 Shadowsocks

### 1.1 Regular

#### 1.1.1 Download release

```bash
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/<version>/shadowsocks-<version>.x86_64-unknown-linux-gnu.tar.xz
```

#### 1.1.2 Decompression

```bash
tar -xvf shadowsocks-<version>.x86_64-unknown-linux-gnu.tar.xz
```

#### 1.1.3 Enter directory

```bash
cd shadowsocks-<version>.x86_64-unknown-linux-gnu
```

#### 1.1.4 Client

##### Start local client with configuration file

```bash
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json
```

##### Socks5 Local client

```bash
# Pass all parameters via command line
sslocal -b "127.0.0.1:1080" -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "v2ray-plugin" --plugin-opts "server;tls;host=github.com"

# Pass server with SIP002 URL
sslocal -b "127.0.0.1:1080" --server-url "ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ@127.0.0.1:8388/?plugin=v2ray-plugin%3Bserver%3Btls%3Bhost%3Dgithub.com"
```

##### HTTP Local client

```bash
sslocal -b "127.0.0.1:3128" --protocol http -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty"
```

All parameters are the same as Socks5 client, except `--protocol http`.

#### 1.1.5 Server

##### Start server with configuration file

```bash
# Read server configuration from file
ssserver -c /path/to/shadowsocks.json
```

##### Start server

```bash
# Pass all parameters via command line
ssserver -s "[::]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "v2ray-plugin" --plugin-opts "server;tls;host=github.com"
```

### 1.2 Docker

This project provided Docker images for the `linux/i386` and `linux/amd64` and `linux/arm64/v8` architectures.

> ⚠️ **Docker containers do not have access to IPv6 by default**: Make sure to disable IPv6 Route in the client or [enable IPv6 access to docker containers](https://docs.docker.com/config/daemon/ipv6/#use-ipv6-for-the-default-bridge-network).

#### 1.2.1 Pull from GitHub Container Registry

Docker will pull the image of the appropriate architecture from our [GitHub Packages](https://github.com/orgs/shadowsocks/packages?repo_name=shadowsocks-rust).

```bash
docker pull ghcr.io/shadowsocks/sslocal-rust:v1.21.2
docker pull ghcr.io/shadowsocks/ssserver-rust:v1.21.2
```

#### 1.2.2 Build on the local machine（Optional）

If you want to build the Docker image yourself, you need to use the [BuildX](https://docs.docker.com/buildx/working-with-buildx/).

```bash
docker buildx build -t shadowsocks/ssserver-rust:v1.21.2 -t shadowsocks/ssserver-rust:v1.21.2 --target ssserver .
docker buildx build -t shadowsocks/sslocal-rust:v1.21.2 -t shadowsocks/sslocal-rust:v1.21.2 --target sslocal .
```

#### 1.2.3 Run the container

You need to mount the configuration file into the container and create an external port map for the container to connect to it.

```bash
docker run -d \
    --name sslocal-rust \
    --restart always \
    -p 1080:1080/tcp \
    -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
    ghcr.io/shadowsocks/sslocal-rust:v1.21.2

docker run -d \
    --name ssserver-rust \
    --restart always \
    -p 8388:8388/tcp \
    -p 8388:8388/udp \
    -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
    ghcr.io/shadowsocks/ssserver-rust:v1.21.2
```

### 1.3 Configuration

```json
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

### 1.4 Usage

#### 1.4.1 Proxychains

##### Install proxychains

```bash
pacman -S proxychains
```

##### Add proxychains config

```sh
# /etc/proxychains.conf

socks5 127.0.0.1 1080
```

##### Using in terminal

```bash
proxychains curl www.example.com
```

#### 1.4.2 Privoxy

##### Install privoxy

```bash
pacman -S privoxy
```

##### Add privoxy config

```sh
# /etc/privoxy/config

forward-socks5 / 127.0.0.1:1080 .
listen-address  127.0.0.1:8118
```

##### Start browser with privoxy

```bash
systemctl start privoxy
chromium --proxy-server="http://127.0.0.1:8118"
```

## 2 ShadowsocksR

### 2.1 Regular

#### 2.1.1 Obtain source code

```bash
git clone -b manyuser https://github.com/shadowsocksr-backup/shadowsocksr.git
```

#### 2.1.2 Enter subdirectory

```bash
cd shadowsocksr/shadowsocks
```

#### 2.1.3 Running via command line

```bash
python local.py -s <server_ip> \
    -p <port> \
    -k <keyphrase> \
    -m <encryption> \
    -o <obfus> \
    -O <protocol> \
    -l <local_port>
```

#### 2.1.4 Running via configuration file

```bash
python local.py -c /etc/shadowsocks.json
```

#### 2.1.5 Stop or restart the daemon

```bash
python local.py -d stop/restart
```

### 2.2 Docker

#### 2.2.1 Pull from Container Registry

```bash
docker pull breakwa11/shadowsocksr:manyuser
```

#### 2.2.2 Running in docker

```bash
docker run -d \
    --name ssrlocal \
    --restart always \
    -p 1080:1080/tcp \
    -v /path/to/config.json:/etc/shadowsocks.json \
    breakwa11/shadowsocksr:manyuser python local.py -c /etc/shadowsocks.json
```

### 2.3 Configuration

```json
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,

    "password": "m",
    "method": "aes-128-ctr",
    "protocol": "auth_aes128_md5",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth_compatible",
    "obfs_param": "",
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```

## 3 Clash

### 3.1 Docker

#### 3.1.1 Pull from Container Registry

```bash
docker pull dreamacro/clash:v1.18.0
```

#### 3.1.2 Run the container

```bash
docker run -d \
    --name clash \
    --restart always \
    -p 7890:7890 \
    -p 7891:7891 \
    -p 7892:7892 \
    -p 9090:9090 \
    -p 5353:5353/udp \
    -p 5353:5353/tcp \
    -v /path/to/config.yaml:/root/.config/clash/config.yaml \
    dreamacro/clash:v1.18.0
```

### 3.2 Configuration

```yaml
# port of HTTP
port: 7890

# port of SOCKS5
socks-port: 7891

# redir port for Linux and macOS
# redir-port: 7892

allow-lan: false

# Only applicable when setting allow-lan to true
# "*": bind all IP addresses
# 192.168.122.11: bind a single IPv4 address
# "[aaaa::a8aa:ff:fe09:57d8]": bind a single IPv6 address
# bind-address: "*"

# rule / global / direct (default is rule)
mode: rule

# set log level to stdout (default is info)
# info / warning / error / debug / silent
log-level: info

# RESTful API for clash
external-controller: 127.0.0.1:9090

# you can put the static web resource (such as clash-dashboard) to a directory, and clash would serve in `${API}/ui`
# input is a relative path to the configuration directory or an absolute path
# external-ui: folder

# Secret for RESTful API (Optional)
# secret: ""

# experimental feature
experimental:
  ignore-resolve-fail: true # ignore dns resolve fail, default value is true
  # interface-name: en0 # outbound interface name

# authentication of local SOCKS5/HTTP(S) server
# authentication:
#  - "user1:pass1"
#  - "user2:pass2"

# # hosts, support wildcard (e.g. *.clash.dev Even *.foo.*.example.com)
# # static domain has a higher priority than wildcard domain (foo.example.com > *.example.com > .example.com)
# # +.foo.com equal .foo.com and foo.com
# hosts:
#   '*.clash.dev': 127.0.0.1
#   '.dev': 127.0.0.1
#   'alpha.clash.dev': '::1'
#   '+.foo.dev': 127.0.0.1

# dns:
  # enable: true # set true to enable dns (default is false)
  # ipv6: false # default is false
  # listen: 0.0.0.0:5353
  # # default-nameserver: # resolve dns nameserver host, should fill pure IP
  # #   - 114.114.114.114
  # #   - 8.8.8.8
  # enhanced-mode: redir-host # or fake-ip
  # # fake-ip-range: 198.18.0.1/16 # if you don't know what it is, don't change it
  # fake-ip-filter: # fake ip white domain list
  #   - '*.lan'
  #   - localhost.ptlogin2.qq.com
  # nameserver:
  #   - 114.114.114.114
  #   - tls://dns.rubyfish.cn:853 # dns over tls
  #   - https://1.1.1.1/dns-query # dns over https
  # fallback: # concurrent request with nameserver, fallback used when GEOIP country isn't CN
  #   - tcp://1.1.1.1
  # fallback-filter:
  #   geoip: true # default
  #   ipcidr: # ips in these subnets will be considered polluted
  #     - 240.0.0.0/4

proxies:
  # shadowsocks
  # The supported ciphers(encrypt methods):
  #   aes-128-gcm aes-192-gcm aes-256-gcm
  #   aes-128-cfb aes-192-cfb aes-256-cfb
  #   aes-128-ctr aes-192-ctr aes-256-ctr
  #   rc4-md5 chacha20-ietf xchacha20
  #   chacha20-ietf-poly1305 xchacha20-ietf-poly1305
  - name: "ss1"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    # udp: true

  # old obfs configuration format remove after prerelease
  - name: "ss2"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    plugin: obfs
    plugin-opts:
      mode: tls # or http
      # host: bing.com

  - name: "ss3"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    plugin: v2ray-plugin
    plugin-opts:
      mode: websocket # no QUIC now
      # tls: true # wss
      # skip-cert-verify: true
      # host: bing.com
      # path: "/"
      # mux: true
      # headers:
      #   custom: value

  # vmess
  # cipher support auto/aes-128-gcm/chacha20-poly1305/none
  - name: "vmess"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # tls: true
    # skip-cert-verify: true
    # servername: example.com # priority over wss host
    # network: ws
    # ws-path: /path
    # ws-headers:
    #   Host: v2ray.com
  
  - name: "vmess-http"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # network: http
    # http-opts:
    #   # method: "GET"
    #   # path:
    #   #   - '/'
    #   #   - '/video'
    #   # headers:
    #   #   Connection:
    #   #     - keep-alive

  # socks5
  - name: "socks"
    type: socks5
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true
    # skip-cert-verify: true
    # udp: true

  # http
  - name: "http"
    type: http
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true # https
    # skip-cert-verify: true

  # snell
  - name: "snell"
    type: snell
    server: server
    port: 44046
    psk: yourpsk
    # obfs-opts:
      # mode: http # or tls
      # host: bing.com

  # trojan
  - name: "trojan"
    type: trojan
    server: server
    port: 443
    password: yourpsk
    # udp: true
    # sni: example.com # aka server name
    # alpn:
    #   - h2
    #   - http/1.1
    # skip-cert-verify: true

proxy-groups:
  # relay chains the proxies. proxies shall not contain a relay. No UDP support.
  # Traffic: clash <-> http <-> vmess <-> ss1 <-> ss2 <-> Internet
  - name: "relay"
    type: relay
    proxies:
      - http
      - vmess
      - ss1
      - ss2

  # url-test select which proxy will be used by benchmarking speed to a URL.
  - name: "auto"
    type: url-test
    proxies:
      - ss1
      - ss2
      - vmess1
    # tolerance: 150
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

  # fallback select an available policy by priority. The availability is tested by accessing an URL, just like an auto url-test group.
  - name: "fallback-auto"
    type: fallback
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

  # load-balance: The request of the same eTLD will be dial on the same proxy.
  - name: "load-balance"
    type: load-balance
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

  # select is used for selecting proxy or proxy group
  # you can use RESTful API to switch proxy, is recommended for use in GUI.
  - name: Proxy
    type: select
    proxies:
      - ss1
      - ss2
      - vmess1
      - auto
  
  - name: UseProvider
    type: select
    use:
      - provider1
    proxies:
      - Proxy
      - DIRECT

proxy-providers:
  provider1:
    type: http
    url: "url"
    interval: 3600
    path: ./hk.yaml
    health-check:
      enable: true
      interval: 600
      url: http://www.gstatic.com/generate_204
  test:
    type: file
    path: /test.yaml
    health-check:
      enable: true
      interval: 36000
      url: http://www.gstatic.com/generate_204

rules:
  - DOMAIN-SUFFIX,google.com,auto
  - DOMAIN-KEYWORD,google,auto
  - DOMAIN,google.com,auto
  - DOMAIN-SUFFIX,ad.com,REJECT
  # rename SOURCE-IP-CIDR and would remove after prerelease
  - SRC-IP-CIDR,192.168.1.201/32,DIRECT
  # optional param "no-resolve" for IP rules (GEOIP IP-CIDR)
  - IP-CIDR,127.0.0.0/8,DIRECT
  - GEOIP,CN,DIRECT
  - DST-PORT,80,DIRECT
  - SRC-PORT,7777,DIRECT
  # FINAL would remove after prerelease
  # you also can use `FINAL,Proxy` or `FINAL,,Proxy` now
  - MATCH,auto
```

### 3.3 Usage

#### 3.3.1 Proxy

```bash
export https_proxy=http://127.0.0.1:7890
export http_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7891
```

## 4 V2ray

### 4.1 Regular

#### 4.1.1 Obtain binaries file

```bash
wget https://github.com/v2fly/v2ray-core/releases/download/<version>/v2ray-linux-64.zip
```

#### 4.1.2 Decompression

```bash
unzip v2ray-linux-64.zip
```

#### 4.1.3 Enter subdirectory

```bash
cd v2ray-linux-64
```

#### 4.1.4 Running via command line

```bash
v2ray run -c /etc/v2ray/config.json
```

### 4.2 Docker

#### 4.2.1 Pull from Container Registry

```bash
docker pull v2fly/v2fly-core:v5.16.1
```

#### 4.2.2 Run the container

```bash
docker run -d \
    --name v2ray \
    -p 10086:10086 \
    -v /path/to/config.json:/etc/v2ray/config.json \
    v2fly/v2fly-core:v5.16.1 run -c /etc/v2ray/config.json
```

### 4.3 Configuration

#### 4.3.1 Server

```json
{
    "inbounds": [
        {
            "port": 10086, // 服务器监听端口
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
                    }
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```

#### 4.3.2 Client

```json
{
    "inbounds": [
        {
            "port": 1080, // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
            "listen": "127.0.0.1",
            "protocol": "socks",
            "settings": {
                "udp": true
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "server", // 服务器地址，请修改为你自己的服务器 ip 或域名
                        "port": 10086, // 服务器端口
                        "users": [
                            {
                                "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
                            }
                        ]
                    }
                ]
            }
        },
        {
            "protocol": "freedom",
            "tag": "direct"
        }
    ],
    "routing": {
        "domainStrategy": "IPOnDemand",
        "rules": [
            {
                "type": "field",
                "ip": [
                    "geoip:private"
                ],
                "outboundTag": "direct"
            }
        ]
    }
}
```