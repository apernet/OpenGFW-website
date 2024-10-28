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

其中 `config.yaml` 是配置文件，`rules.yaml` 是规则文件。

#### pcap 文件回放模式

```shell
./OpenGFW -p your.pcap -c config.yaml rules.yaml
```

在 pcap 模式下，规则中的所有动作都没有任何效果。此模式主要用于调试。

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
  queueNum: 100 # (6)!
  table: opengfw # (7)!
  connMarkAccept: 1001 # (8)!
  connMarkDrop: 1002 # (9)!
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

# 指定的 geoip/geosite 档案路径
# 如果未设置，将自动从 https://github.com/Loyalsoldier/v2ray-rules-dat 下载
# ruleset:
#   geoip: geoip.dat
#   geosite: geosite.dat

replay:
  realtime: false # (5)!
```

1. 如果想在 FORWARD 链上运行（如在路由器上），设置为 false
2. 如果想为被阻断的 TCP 连接发送 RST，设置为 true。**仅在 local=false 时有效**
3. 建议不超过 CPU 核心数
4. 一个连接多久没有数据传输后会被认为是死连接。TCP 重组的连接池会以每分钟一次的频率清理死连接
5. 如果希望以实时速度（pcap 文件中的时间戳）回放 pcap 中的数据包（而不是以能处理的最快速度），设置为 true
6. nfqueue 队列序号
7. nftables 表名
8. 放行连接的 connmark 值
9. 阻断连接的 connmark 值
