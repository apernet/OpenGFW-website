---
title: 内置函数
---

除了 [expr 本身内置的函数以外](https://expr-lang.org/docs/language-definition)，我们还提供了一些额外的内置函数，可以在表达式中使用：

### `cidr`

```
cidr(ip: string, cidr: string) -> bool
```

检查一个 IP 地址是否在一个 CIDR 范围内。示例：

```yaml
- name: block cidr
  action: block
  expr: cidr(string(ip.dst), "192.168.0.0/16")
```

### `geoip`

```
geoip(ip: string, country: string) -> bool
```

检查一个 IP 地址是否来自一个国家，使用来自 https://github.com/Loyalsoldier/v2ray-rules-dat 的数据。

示例：

```yaml
- name: block CN geoip
  action: block
  expr: geoip(string(ip.dst), "cn")
```

### `geosite`

```
geosite(domain: string, category: string) -> bool
```

检查一个域名是否属于一个特定的类别，使用来自 https://github.com/Loyalsoldier/v2ray-rules-dat 的数据。

示例：

```yaml
- name: block bilibili geosite
  action: block
  expr: geosite(string(tls?.req?.sni), "bilibili")
```

### `lookup`

```
lookup(domain: string) -> list<string>
lookup(domain: string, server: string) -> list<string>
```

对指定的域名进行 DNS 查询，返回 IP 地址列表（同时包括 A 和 AAAA 记录）。如果未指定服务器地址，则使用系统默认的 DNS。此操作使用标准 DNS 协议（不是 DNS over TLS、DNS over HTTPS 等），且服务器地址必须同时包括 IP 地址和端口（如 `8.8.8.8:53`）。

示例：

```yaml
- name: SNI mismatch
  log: true
  expr: tls?.req?.sni != nil && ip.dst not in lookup(tls.req.sni)
```
