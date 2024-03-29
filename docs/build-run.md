---
title: Build & Run
hide:
  - navigation
---

### Build

```shell
export CGO_ENABLED=0
go build
```

### Run

```shell
export OPENGFW_LOG_LEVEL=debug
./OpenGFW -c config.yaml rules.yaml
```

#### OpenWrt

OpenGFW has been tested to work on OpenWrt 23.05 (other versions should also work, just not verified).

Install the dependencies:

```shell
opkg install nftables kmod-nft-queue kmod-nf-conntrack-netlink
```

### Example config

```yaml
io:
  queueSize: 1024
  rcvBuf: 4194304
  sndBuf: 4194304
  local: true # set to false if you want to run OpenGFW on FORWARD chain
  rst: false # set to true if you want to send RST for blocked TCP connections, local=false only

workers:
  count: 4
  queueSize: 16
  tcpMaxBufferedPagesTotal: 4096
  tcpMaxBufferedPagesPerConn: 64
  udpMaxStreams: 4096
# The path to load specific local geoip/geosite db files.
# If not set, they will be automatically downloaded from https://github.com/Loyalsoldier/v2ray-rules-dat
# geo:
#   geoip: geoip.dat
#   geosite: geosite.dat
```

### Example rules

[Analyzer properties](analyzers.md)

For syntax of the expression language, please refer
to [Expr Language Definition](https://expr-lang.org/docs/language-definition).

```yaml
# A rule must have at least one of "action" or "log" field set.
- name: log horny people
  log: true
  expr: let sni = string(tls?.req?.sni); sni contains "porn" || sni contains "hentai"

- name: block v2ex http
  action: block
  expr: string(http?.req?.headers?.host) endsWith "v2ex.com"

- name: block v2ex https
  action: block
  expr: string(tls?.req?.sni) endsWith "v2ex.com"

- name: block v2ex quic
  action: block
  expr: string(quic?.req?.sni) endsWith "v2ex.com"

- name: block and log shadowsocks
  action: block
  log: true
  expr: fet != nil && fet.yes

- name: block trojan
  action: block
  expr: trojan != nil && trojan.yes

- name: v2ex dns poisoning
  action: modify
  modifier:
    name: dns
    args:
      a: "0.0.0.0"
      aaaa: "::"
  expr: dns != nil && dns.qr && any(dns.questions, {.name endsWith "v2ex.com"})

- name: block google socks
  action: block
  expr: string(socks?.req?.addr) endsWith "google.com" && socks?.req?.port == 80

- name: block wireguard by handshake response
  action: drop
  expr: wireguard?.handshake_response?.receiver_index_matched == true

- name: block bilibili geosite
  action: block
  expr: geosite(string(tls?.req?.sni), "bilibili")

- name: block CN geoip
  action: block
  expr: geoip(string(ip.dst), "cn")

- name: block cidr
  action: block
  expr: cidr(string(ip.dst), "192.168.0.0/16")
```

#### Supported actions

- `allow`: Allow the connection, no further processing.
- `block`: Block the connection, no further processing.
- `drop`: For UDP, drop the packet that triggered the rule, continue processing future packets in the same flow. For
  TCP, same as `block`.
- `modify`: For UDP, modify the packet that triggered the rule using the given modifier, continue processing future
  packets in the same flow. For TCP, same as `allow`.
