# ipwatch

内网设备在线状态追踪器：周期性探测局域网网段，把"谁在/谁掉线/掉多久"写进 SQLite，
自带 Web 仪表盘。独立于 [starfleet](https://github.com/snail2sky/starfleet)，可单独使用，
也可以交给 starfleet 托管运行。

```
   ┌────────────┐   ARP/ICMP 探测    ┌─────────────┐
   │  ipwatch   │ ─────────────────▶ │  LAN 网段    │
   │  (sweeper) │                    │ 192.168.1.0/24│
   └─────┬──────┘                    └─────────────┘
         │ 每轮 sightings
         ▼
   ┌────────────┐   状态机(在线/离线) ┌─────────────┐
   │  tracker   │ ─────────────────▶ │ SQLite 存档 │
   └─────┬──────┘                    └─────────────┘
         │ HTTP / WS
         ▼
   ┌────────────┐
   │ Web 仪表盘 │  :8095
   └────────────┘
```

## 用法

```bash
ipwatch -subnet 192.168.1.0/24 -db ipwatch.db          # 开始记录（默认 web :8095）
ipwatch -db ipwatch.db -summary                        # 设备列表 + 今日掉线片段
ipwatch -db ipwatch.db -once                           # 只扫一轮就退出
ipwatch -db ipwatch.db -interval 15s -retain 720h      # 探测周期 / 原始数据保留期
```

Web 仪表盘：`http://<host>:8095`（设备、在线趋势、掉线片段）。

数据库是普通 SQLite：

```bash
sqlite3 ipwatch.db 'SELECT * FROM presence ORDER BY online_at DESC LIMIT 20'
```

## 交给 starfleet 托管（可选）

```yaml
processes:
  ipwatch:
    command: "/usr/local/bin/ipwatch -subnet 192.168.1.0/24 -db /data0/apps/ipwatch/ipwatch.db"
    directory: "/data0/apps/ipwatch"
    user: orangepi
    autostart: true
```

## 安装

```bash
# 源码
go build -trimpath -ldflags "-s -w" -o ipwatch .

# 或从 release 下载对应平台的压缩包
```

---
*Go · SQLite(纯Go, 无需 CGO) · 单二进制*
