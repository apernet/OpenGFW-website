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
