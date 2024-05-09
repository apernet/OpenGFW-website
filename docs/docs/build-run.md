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

#### pcap file replay mode

```shell
./OpenGFW -p your.pcap -c config.yaml rules.yaml
```

In pcap mode, none of the actions in the rules have any effect. This mode is mainly for debugging.

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
  local: true # (1)!
  rst: false # (2)!

workers:
  count: 4 # (3)!
  queueSize: 64
  tcpMaxBufferedPagesTotal: 65536
  tcpMaxBufferedPagesPerConn: 16
  tcpTimeout: 10m # (4)!
  udpMaxStreams: 4096

# The path to load specific local geoip/geosite db files.
# If not set, they will be automatically downloaded from https://github.com/Loyalsoldier/v2ray-rules-dat
# ruleset:
#   geoip: geoip.dat
#   geosite: geosite.dat

replay:
  realtime: false # (5)!
```

1. Set to false if you want to run OpenGFW on FORWARD chain (e.g. on a router)
2. Set to true if you want to send RST for blocked TCP connections, **local=false only**
3. Recommended to be no more than the number of CPU cores
4. How long a connection is considered dead when no data is being transferred. Dead connections are purged from TCP reassembly pools once per minute.
5. Set to true if you want to replay the packets in the pcap file in "real time" (instead of as fast as possible)
