---
title: 分析器
---

分析器是 OpenGFW 的重要组件之一，作用是分析连接，检查是否是支持的协议，并从该连接中提取信息，作为提供给规则引擎的属性，以便与用户提供的规则进行匹配。OpenGFW 会自动分析提供的规则中引用了哪些分析器，仅启用需要的分析器。

本文档列出了每个分析器提供的属性。所有列出的属性都可以在规则中使用。

!!! tip

    许多分析器并非一次性提供全部属性，而是会随着传输进行逐步增加/更新字段。例如，当一个 HTTP 连接发送了请求但还没有收到响应时，HTTP 分析器只会有 `req` 部分而没有 `resp`。每个连接会在任何属性发生变化时匹配一次规则。规则需要能正确处理需要的字段为 `nil` 的情况。（如 `http != nil && http.resp != nil` 或利用 `?` 操作符 `http?.resp`）

## 内置

每个连接都会有以下内置属性：

```json
{
  "id": 123456,
  "proto": "tcp", // tcp, udp
  "ip": {
    "src": "123.123.123.123",
    "dst": "7.8.9.0"
  },
  "port": {
    "src": 1234,
    "dst": 80
  }
}
```

阻止到 `8.8.8.8` 的 UDP 流量：

```yaml
- name: Block 8.8.8.8 UDP
  action: block
  expr: ip.dst == "8.8.8.8" && proto == "udp"
```

## DNS (TCP & UDP)

对于请求：

```json
{
  "dns": {
    "aa": false,
    "id": 41953,
    "opcode": 0,
    "qr": false,
    "questions": [
      {
        "class": 1,
        "name": "www.google.com",
        "type": 1
      }
    ],
    "ra": false,
    "rcode": 0,
    "rd": true,
    "tc": false,
    "z": 0
  }
}
```

对于相应：

```json
{
  "dns": {
    "aa": false,
    "answers": [
      {
        "a": "142.251.32.36",
        "class": 1,
        "name": "www.google.com",
        "ttl": 255,
        "type": 1
      }
    ],
    "id": 41953,
    "opcode": 0,
    "qr": true,
    "questions": [
      {
        "class": 1,
        "name": "www.google.com",
        "type": 1
      }
    ],
    "ra": true,
    "rcode": 0,
    "rd": true,
    "tc": false,
    "z": 0
  }
}
```

丢弃 `www.google.com` 的 DNS 请求：

```yaml
- name: Block Google DNS
  action: drop
  expr: dns != nil && !dns.qr && any(dns.questions, {.name == "www.google.com"})
```

## FET (全加密连接)

请阅读论文 https://www.usenix.org/system/files/usenixsecurity23-wu-mingshi.pdf 以获取更多信息。

```json
{
  "fet": {
    "ex1": 3.7560976,
    "ex2": true,
    "ex3": 0.9512195,
    "ex4": 39,
    "ex5": false,
    "yes": false
  }
}
```

屏蔽全加密连接：

```yaml
- name: Block suspicious proxy traffic
  action: block
  expr: fet != nil && fet.yes
```

## HTTP

```json
{
  "http": {
    "req": {
      "headers": {
        "accept": "*/*",
        "host": "ipinfo.io",
        "user-agent": "curl/7.81.0"
      },
      "method": "GET",
      "path": "/",
      "version": "HTTP/1.1"
    },
    "resp": {
      "headers": {
        "access-control-allow-origin": "*",
        "content-length": "333",
        "content-type": "application/json; charset=utf-8",
        "date": "Wed, 24 Jan 2024 05:41:44 GMT",
        "referrer-policy": "strict-origin-when-cross-origin",
        "server": "nginx/1.24.0",
        "strict-transport-security": "max-age=2592000; includeSubDomains",
        "via": "1.1 google",
        "x-content-type-options": "nosniff",
        "x-envoy-upstream-service-time": "2",
        "x-frame-options": "SAMEORIGIN",
        "x-xss-protection": "1; mode=block"
      },
      "status": 200,
      "version": "HTTP/1.1"
    }
  }
}
```

屏蔽对 `ipinfo.io` 的 HTTP 请求：

```yaml
- name: Block ipinfo.io HTTP
  action: block
  expr: http != nil && http.req != nil && http.req.headers != nil && http.req.headers.host == "ipinfo.io"
```

## SSH

```json
{
  "ssh": {
    "server": {
      "comments": "Ubuntu-3ubuntu0.6",
      "protocol": "2.0",
      "software": "OpenSSH_8.9p1"
    },
    "client": {
      "comments": "IMHACKER",
      "protocol": "2.0",
      "software": "OpenSSH_8.9p1"
    }
  }
}
```

屏蔽 SSH：

```yaml
- name: Block SSH
  action: block
  expr: ssh != nil
```

## TLS

```json
{
  "tls": {
    "req": {
      "alpn": ["h2", "http/1.1"],
      "ciphers": [
        4866, 4867, 4865, 49196, 49200, 159, 52393, 52392, 52394, 49195, 49199,
        158, 49188, 49192, 107, 49187, 49191, 103, 49162, 49172, 57, 49161,
        49171, 51, 157, 156, 61, 60, 53, 47, 255
      ],
      "compression": "AA==",
      "random": "UqfPi+EmtMgusILrKcELvVWwpOdPSM/My09nPXl84dg=",
      "session": "jCTrpAzHpwrfuYdYx4FEjZwbcQxCuZ52HGIoOcbw1vA=",
      "sni": "ipinfo.io",
      "supported_versions": [772, 771],
      "version": 771,
      "ech": true
    },
    "resp": {
      "cipher": 4866,
      "compression": 0,
      "random": "R/Cy1m9pktuBMZQIHahD8Y83UWPRf8j8luwNQep9yJI=",
      "session": "jCTrpAzHpwrfuYdYx4FEjZwbcQxCuZ52HGIoOcbw1vA=",
      "supported_versions": 772,
      "version": 771
    }
  }
}
```

屏蔽对 `ipinfo.io` 的 TLS 请求：

```yaml
- name: Block ipinfo.io TLS
  action: block
  expr: tls != nil && tls.req != nil && tls.req.sni == "ipinfo.io"
```

## QUIC

QUIC 解析器的格式与 TLS 一样，但是目前只支持请求 (req) 部分：

```json
{
  "quic": {
    "req": {
      "alpn": ["h3"],
      "ciphers": [4865, 4866, 4867],
      "compression": "AA==",
      "ech": true,
      "random": "FUYLceFReLJl9dRQ0HAus7fi2ZGuKIAApF4keeUqg00=",
      "session": "",
      "sni": "quic.rocks",
      "supported_versions": [772],
      "version": 771
    }
  }
}
```

屏蔽对 `quic.rocks` 的 QUIC 请求：

```yaml
- name: Block quic.rocks QUIC
  action: block
  expr: quic != nil && quic.req != nil && quic.req.sni == "quic.rocks"
```

## Trojan (代理协议)

```json
{
  "trojan": {
    "seq": [680, 4514, 293],
    "yes": true
  }
}
```

屏蔽 Trojan 连接：

```yaml
- name: Block Trojan
  action: block
  expr: trojan != nil && trojan.yes
```

## SOCKS

SOCKS4:

```json
{
  "socks": {
    "version": 4,
    "req": {
      "cmd": 1,
      "addr_type": 1, // same as socks5
      "addr": "1.1.1.1",
      // for socks4a
      // "addr_type": 3,
      // "addr": "google.com",
      "port": 443,
      "auth": {
        "user_id": "user"
      }
    },
    "resp": {
      "rep": 90, // 0x5A(90) granted
      "addr_type": 1,
      "addr": "1.1.1.1",
      "port": 443
    }
  }
}
```

SOCKS5 无验证:

```json
{
  "socks": {
    "version": 5,
    "req": {
      "cmd": 1, // 0x01: connect, 0x02: bind, 0x03: udp
      "addr_type": 3, // 0x01: ipv4, 0x03: domain, 0x04: ipv6
      "addr": "google.com",
      "port": 80,
      "auth": {
        "method": 0 // 0x00: no auth, 0x02: username/password
      }
    },
    "resp": {
      "rep": 0, // 0x00: success
      "addr_type": 1, // 0x01: ipv4, 0x03: domain, 0x04: ipv6
      "addr": "198.18.1.31",
      "port": 80,
      "auth": {
        "method": 0 // 0x00: no auth, 0x02: username/password
      }
    }
  }
}
```

SOCKS5 带验证:

```json
{
  "socks": {
    "version": 5,
    "req": {
      "cmd": 1, // 0x01: connect, 0x02: bind, 0x03: udp
      "addr_type": 3, // 0x01: ipv4, 0x03: domain, 0x04: ipv6
      "addr": "google.com",
      "port": 80,
      "auth": {
        "method": 2, // 0x00: no auth, 0x02: username/password
        "username": "user",
        "password": "pass"
      }
    },
    "resp": {
      "rep": 0, // 0x00: success
      "addr_type": 1, // 0x01: ipv4, 0x03: domain, 0x04: ipv6
      "addr": "198.18.1.31",
      "port": 80,
      "auth": {
        "method": 2, // 0x00: no auth, 0x02: username/password
        "status": 0 // 0x00: success, 0x01: failure
      }
    }
  }
}
```

屏蔽到 `google.com:80` 的 SOCKS 请求，以及带有用户名 `foobar` 的 SOCKS 请求：

```yaml
- name: Block SOCKS google.com:80
  action: block
  expr: string(socks?.req?.addr) endsWith "google.com" && socks?.req?.port == 80

- name: Block SOCKS user foobar
  action: block
  expr: socks?.req?.auth?.method == 2 && socks?.req?.auth?.username == "foobar"
```

## WireGuard

```json
{
  "wireguard": {
    "message_type": 1, // 0x1: handshake_initiation, 0x2: handshake_response, 0x3: packet_cookie_reply, 0x4: packet_data
    "handshake_initiation": {
      "sender_index": 0x12345678
    },
    "handshake_response": {
      "sender_index": 0x12345678,
      "receiver_index": 0x87654321,
      "receiver_index_matched": true
    },
    "packet_data": {
      "receiver_index": 0x12345678,
      "receiver_index_matched": true
    },
    "packet_cookie_reply": {
      "receiver_index": 0x12345678,
      "receiver_index_matched": true
    }
  }
}
```

屏蔽 WireGuard：

```yaml
# 误伤: 高
- name: Block all WireGuard-like traffic
  action: block
  expr: wireguard != nil

# 误伤: 中
- name: Block WireGuard by handshake_initiation
  action: drop
  expr: wireguard?.handshake_initiation != nil

# 误伤: 低
- name: Block WireGuard by handshake_response
  action: drop
  expr: wireguard?.handshake_response?.receiver_index_matched == true

# 误伤: 极低
- name: Block WireGuard by packet_data
  action: block
  expr: wireguard?.packet_data?.receiver_index_matched == true
```
