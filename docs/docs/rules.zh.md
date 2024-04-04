---
title: 规则文件
---

规则文件是规则的集合，用于定义对匹配到的连接进行何种操作。一个规则的基本格式如下：

```yaml
- name: block v2ex https
  action: block
  log: true
  expr: string(tls?.req?.sni) endsWith "v2ex.com"
```

其中 `name` 是规则的名称，`action` 是进行的动作，`log` 是是否记录日志，`expr` 是匹配表达式。**`action` 和 `log` 中必须至少存在一个。**

表达式采用的是 [Expr 语言](https://expr-lang.org/)，语法请参考其 [语法定义](https://expr-lang.org/docs/language-definition)。

可供匹配的数据来自分析器，请参考 [分析器文档](analyzers.md)。

### 支持的动作 (action)

| 动作     | TCP                          | UDP                                                            |
| -------- | ---------------------------- | -------------------------------------------------------------- |
| `allow`  | 放行连接，不再处理后续的包。 | 放行连接，不再处理后续的包。                                   |
| `block`  | 阻断连接，不再处理后续的包。 | 阻断连接，不再处理后续的包。                                   |
| `drop`   | 效果同 `block`。             | 丢弃触发规则的包，但继续处理同一流中的后续包。                 |
| `modify` | 效果同 `allow`。             | 用指定的修改器修改触发规则的包，然后继续处理同一流中的后续包。 |

### 规则样例

#### 记录 SNI 中包含特定关键字的连接

```yaml
- name: log horny people
  log: true
  expr: let sni = string(tls?.req?.sni); sni contains "porn" || sni contains "hentai"
```

#### 阻断访问 v2ex.com 的 HTTP/HTTPS/QUIC 连接

```yaml
- name: block v2ex http
  action: block
  expr: string(http?.req?.headers?.host) endsWith "v2ex.com"

- name: block v2ex https
  action: block
  expr: string(tls?.req?.sni) endsWith "v2ex.com"

- name: block v2ex quic
  action: block
  expr: string(quic?.req?.sni) endsWith "v2ex.com"
```

#### 阻断并记录 Shadowsocks, VMess, Trojan 连接

```yaml
- name: block shadowsocks and vmess
  action: block
  log: true
  expr: fet != nil && fet.yes

- name: block trojan
  action: block
  log: true
  expr: trojan != nil && trojan.yes
```

#### 将 v2ex.com 域名 DNS 污染到 0.0.0.0 和 ::

```yaml
- name: v2ex dns poisoning
  action: modify
  modifier:
    name: dns
    args:
      a: "0.0.0.0"
      aaaa: "::"
  expr: dns != nil && dns.qr && any(dns.questions, {.name endsWith "v2ex.com"})
```

#### 阻断 SOCKS 代理访问 google.com:80

```yaml
- name: block google socks
  action: block
  expr: string(socks?.req?.addr) endsWith "google.com" && socks?.req?.port == 80
```

#### 根据握手响应阻断 WireGuard

```yaml
- name: block wireguard by handshake response
  action: drop
  expr: wireguard?.handshake_response?.receiver_index_matched == true
```

#### 根据 GeoSite 数据库阻断 Bilibili 的所有域名

```yaml
- name: block bilibili geosite
  action: block
  expr: geosite(string(tls?.req?.sni), "bilibili")
```

#### 根据 GeoIP 数据库阻断所有目标 IP 为中国的连接

```yaml
- name: block CN geoip
  action: block
  expr: geoip(string(ip.dst), "cn")
```

#### 根据 CIDR 阻断特定 IP 段的连接

```yaml
- name: block cidr
  action: block
  expr: cidr(string(ip.dst), "192.168.0.0/16")
```
