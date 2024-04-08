---
title: Functions
---

In addition to [everything that expr already provides](https://expr-lang.org/docs/language-definition), we also have some extra built-in functions that you can use in your expressions:

### `cidr`

```
cidr(ip: string, cidr: string) -> bool
```

Check if an IP address is in a CIDR range. Example:

```yaml
- name: block cidr
  action: block
  expr: cidr(string(ip.dst), "192.168.0.0/16")
```

### `geoip`

```
geoip(ip: string, country: string) -> bool
```

Check if an IP address belongs to a specific country, using data from https://github.com/Loyalsoldier/v2ray-rules-dat

Example:

```yaml
- name: block CN geoip
  action: block
  expr: geoip(string(ip.dst), "cn")
```

### `geosite`

```
geosite(domain: string, category: string) -> bool
```

Check if a domain belongs to a specific category, using data from https://github.com/Loyalsoldier/v2ray-rules-dat

Example:

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

Perform a DNS lookup for a domain, returns the list of IP addresses (both A and AAAA records) returned by the DNS server. If the server address is not specified, it uses the system default. Note that this uses the standard DNS protocol (not DNS over TLS, DNS over HTTPS, for example), and you must specify both IP and port for the server address (e.g. `8.8.8.8:53`).

Example:

```yaml
- name: SNI mismatch
  log: true
  expr: tls?.req?.sni != nil && ip.dst not in lookup(tls.req.sni)
```
