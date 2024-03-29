---
title: Home
hide:
  - navigation
  - toc
---

<!-- Hack to hide the title -->
<style>
  .md-typeset h1,
  .md-content__button {
    display: none;
  }
</style>

<div style="text-align: center;">
  <img src="/assets/logo_text.png" alt="OpenGFW Logo">
</div>

OpenGFW is your very own DIY [Great Firewall of China](https://en.wikipedia.org/wiki/Great_Firewall), available as a flexible, easy-to-use open source program on Linux. Why let the powers that be have all the fun? It's time to give power to the people and democratize censorship. Bring the thrill of cyber-sovereignty right into your home router and start filtering like a pro - you too can play Big Brother.

!!! warning

    This project is still in very early stages of development. Use at your own risk. We are looking for contributors to help us improve and expand the project.

[:fontawesome-brands-telegram: Telegram Group](https://t.me/OpGFW)

## Features

- Full IP/TCP reassembly, various protocol analyzers
  - HTTP, TLS, QUIC, DNS, SSH, SOCKS4/5, WireGuard, and many more to come
  - "Fully encrypted traffic" detection for Shadowsocks,
    etc. (https://gfw.report/publications/usenixsecurity23/en/)
  - Trojan (proxy protocol) detection
  - [WIP] Machine learning based traffic classification
- Full IPv4 and IPv6 support
- Flow-based multicore load balancing
- Connection offloading
- Powerful rule engine based on [expr](https://github.com/expr-lang/expr)
- Hot-reloadable rules (send `SIGHUP` to reload)
- Flexible analyzer & modifier framework
- Extensible IO implementation (only NFQueue for now)
- [WIP] Web UI

## Use cases

- Ad blocking
- Parental control
- Malware protection
- Abuse prevention for VPN/proxy services
- Traffic analysis (log only mode)
- Help you fulfill your dictatorial ambitions
