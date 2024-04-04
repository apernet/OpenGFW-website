---
title: Build & Run
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

Where `config.yaml` is the config file and `rules.yaml` is the rules file.

#### OpenWrt

OpenGFW has been tested to work on OpenWrt 23.05 (other versions should also work, just not verified).

Install the dependencies:

```shell
opkg install nftables kmod-nft-queue kmod-nf-conntrack-netlink
```

### Config example

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
