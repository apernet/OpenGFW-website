---
title: 构建与运行
---

### 构建

```shell
export CGO_ENABLED=0
go build
```

### 运行

```shell
export OPENGFW_LOG_LEVEL=debug
./OpenGFW -c config.yaml rules.yaml
```

#### OpenWrt

OpenGFW 在 OpenWrt 23.05 上测试可用（其他版本应该也可以，暂时未经验证）。

安装依赖：

```shell
opkg install nftables kmod-nft-queue kmod-nf-conntrack-netlink
```

### 样例配置

```yaml
io:
  queueSize: 1024
  rcvBuf: 4194304
  sndBuf: 4194304
  local: true # 如果需要在 FORWARD 链上运行 OpenGFW，请设置为 false
  rst: false # 是否对要阻断的 TCP 连接发送 RST。仅在 local=false 时有效

workers:
  count: 4
  queueSize: 16
  tcpMaxBufferedPagesTotal: 4096
  tcpMaxBufferedPagesPerConn: 64
  udpMaxStreams: 4096
# 指定的 geoip/geosite 档案路径
# 如果未设置，将自动从 https://github.com/Loyalsoldier/v2ray-rules-dat 下载
# geo:
#   geoip: geoip.dat
#   geosite: geosite.dat
```

### 样例规则

[解析器属性](analyzers.md)

规则的语法请参考 [Expr Language Definition](https://expr-lang.org/docs/language-definition)。

```yaml
# 每条规则必须至少包含 action 或 log 中的一个。
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

#### 支持的 action

| 动作     | TCP                          | UDP                                                            |
| -------- | ---------------------------- | -------------------------------------------------------------- |
| `allow`  | 放行连接，不再处理后续的包。 | 放行连接，不再处理后续的包。                                   |
| `block`  | 阻断连接，不再处理后续的包。 | 阻断连接，不再处理后续的包。                                   |
| `drop`   | 效果同 `block`。             | 丢弃触发规则的包，但继续处理同一流中的后续包。                 |
| `modify` | 效果同 `allow`。             | 用指定的修改器修改触发规则的包，然后继续处理同一流中的后续包。 |
