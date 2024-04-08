---
title: Rules
---

The rule file is a collection of rules used to define what actions to take on matched connections. The format of a rule is as follows:

```yaml
- name: block v2ex https
  action: block
  log: true
  expr: string(tls?.req?.sni) endsWith "v2ex.com"
```

Where `name` is the name of the rule, `action` is the action to take, `log` is whether to print logs, and `expr` is the matching expression. **A rule must have at least one of `action` or `log` field set.**

The expression uses the [Expr language](https://expr-lang.org/), please refer to its [language definition](https://expr-lang.org/docs/language-definition) for syntax.

The data available for matching comes from analyzers, please refer to the [analyzer documentation](analyzers.md).

### Supported actions

| Action   | TCP                                          | UDP                                                                                                                      |
| -------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `allow`  | Allow the connection, no further processing. | Allow the connection, no further processing.                                                                             |
| `block`  | Block the connection, no further processing. | Block the connection, no further processing.                                                                             |
| `drop`   | Same as `block`.                             | Drop the packet that triggered the rule, continue processing future packets in the same flow.                            |
| `modify` | Same as `allow`.                             | Modify the packet that triggered the rule using the given modifier, continue processing future packets in the same flow. |

### Rule examples

#### Log connections with specific keywords in SNI

```yaml
- name: log horny people
  log: true
  expr: let sni = string(tls?.req?.sni); sni contains "porn" || sni contains "hentai"
```

#### Block HTTP/HTTPS/QUIC connections to v2ex.com

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

#### Block and log Shadowsocks, VMess, Trojan connections

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

#### DNS poison v2ex.com to 0.0.0.0 and ::

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

#### Block SOCKS proxy access to google.com:80

```yaml
- name: block google socks
  action: block
  expr: string(socks?.req?.addr) endsWith "google.com" && socks?.req?.port == 80
```

#### Block WireGuard by handshake response

```yaml
- name: block wireguard by handshake response
  action: drop
  expr: wireguard?.handshake_response?.receiver_index_matched == true
```

#### Block all bilibili domains using GeoSite

```yaml
- name: block bilibili geosite
  action: block
  expr: geosite(string(tls?.req?.sni), "bilibili")
```

#### Block all connections to China using GeoIP

```yaml
- name: block CN geoip
  action: block
  expr: geoip(string(ip.dst), "cn")
```

#### Block all connections to a specific CIDR range

```yaml
- name: block cidr
  action: block
  expr: cidr(string(ip.dst), "192.168.0.0/16")
```

#### Block Xray Reality/ShadowTLS connections

How it works: The TLS handshake of protocols like Xray Reality/ShadowTLS "steals" that of real websites, but the destination IP is the proxy server rather than a real IP that belongs to those websites. Therefore, you can check the connection by performing DNS lookups on the SNI domain name; if the destination IP isn't among what's in the DNS records, then the connection can be considered suspicious.

!!! warning

    To minimize false positives, the rule below queries two additional servers besides the system's default DNS. The connection is allowed as long as the destination IP is in one of the results. Be sure to adjust the rule to your actual network environment. If the domain name cannot be resolved, the `lookup` function will fail, causing this rule to fail as well, and the connection won't be blocked.

```yaml
- name: SNI mismatch
  action: block
  expr: tls?.req?.sni != nil && ip.dst not in concat(lookup(tls.req.sni), lookup(tls.req.sni, "1.1.1.1:53"), lookup(tls.req.sni, "8.8.8.8:53"))
```
