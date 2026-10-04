# NanoPi R2S SoftRouter Lab

> NanoPi R2S 上跑 **Nikki (Mihomo)** + **Tailscale Exit Node / 子网路由** 的实战笔记。  
> 从刷机、装不上包、没默认路由、磁盘 99%、Exit「ping 通网页超时」，到最终打通的完整路径。

**硬件**：FriendlyELEC NanoPi R2S（RK3328 / 1GB RAM）  
**系统**：ImmortalWrt 25.12 / iStoreOS（同一类 OpenWrt 系）  
**角色**：接主路由 LAN 的「普通设备」+ 24h 代理保底 + Tailscale 出口

仓库公开，方便自己回顾，也给后来者少踩坑。

---

## 目标架构（最终稳定向）

```text
主路由 (192.168.124.1)
    │
    ├─ 普通设备…
    └─ R2S LAN口 ── 静态 IP 192.168.124.67
           │
           ├─ Nikki / Mihomo（分流 / TUN / 面板）
           └─ Tailscale
                 ├─ Exit Node（异地设备整机出网走家里）
                 └─ Subnet 192.168.124.0/24（访问家里设备）
```

- **不做** 传统旁路由强接管全家（可先当普通设备验证稳定）
- 节点订阅建议压到 **≤256**，1GB 机器容易 OOM
- Overlay 只有约 300MB 时必须清空间或扩容，否则服务会「假死」

---

## 目录

1. [网络：普通设备模式](#1-网络普通设备模式)
2. [Nikki 安装（离线 apk）](#2-nikki-安装离线-apk)
3. [磁盘空间（99% 必读）](#3-磁盘空间99-必读)
4. [Tailscale Exit Node](#4-tailscale-exit-node)
5. [Nikki + Tailscale 共存](#5-nikki--tailscale-共存)
6. [常见故障速查](#6-常见故障速查)
7. [防泄露（通用要点）](#7-防泄露通用要点)
8. [相关链接](#8-相关链接)

---

## 1. 网络：普通设备模式

LAN 口接主路由 LAN，静态地址与主路由同网段。

```sh
uci set network.lan.proto='static'
uci set network.lan.ipaddr='192.168.124.67'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='192.168.124.1'
uci set network.lan.dns='192.168.124.1'
uci set dhcp.lan.ignore='1'
uci commit network
uci commit dhcp
/etc/init.d/network restart
```

**必查默认路由**（缺了就会 `Network unreachable`，装不了任何在线包）：

```sh
ip route
# 必须有：default via 192.168.124.1 dev br-lan

# 若没有，临时：
ip route add default via 192.168.124.1 dev br-lan
```

界面注意：

- 网关填 `192.168.124.1`，不要填成掩码
- 不要残留默认的 `192.168.1.1`
- DHCP 服务器对 lan：**忽略此接口**

---

## 2. Nikki 安装（离线 apk）

架构：`aarch64_generic`（与 ImmortalWrt 25.12 对应包一致）。

### 必装包（不要装 alpha 内核）

- `mihomo-meta-*.apk`
- `nikki-*.apk`
- `luci-app-nikki-*.apk`
- 中文语言包可跳过（装不上也不影响运行）

### 依赖（需先能上网 + `apk update`）

```sh
apk add ip-full curl yq kmod-tun kmod-inet-diag kmod-dummy \
  kmod-nft-socket kmod-nft-tproxy ca-bundle
```

### 本地安装

```sh
cd /tmp
apk add --allow-untrusted /tmp/mihomo-meta-*.apk
apk add --allow-untrusted /tmp/nikki-*.apk
apk add --allow-untrusted /tmp/luci-app-nikki-*.apk

rm -f /tmp/luci-indexcache
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

**不要**把整个 `nikki_….tar.gz` 当单个软件包上传到 LuCI。

运行后在 **服务 → Nikki**：导入订阅（≤256 节点）→ 启用 → Restart。  
面板优先用 Nikki 自带 Dashboard；浏览器 HTTPS 版 Metacubexd 访问 HTTP 的 `external-controller` 会被混合内容拦截。

---

## 3. 磁盘空间（99% 必读）

典型现象：`df` 显示 overlay 约 **291MB 已用 99%**，只剩 2MB。  
Tailscale 状态写不进、apk 失败、服务抽风都可能由此引起。

```sh
df -h
# overlayfs / loop0 ≈ 300MB 是可写层，不是整张 16G TF

# 应急清理
rm -f /tmp/*.apk /tmp/*.tar.gz
rm -rf /tmp/luci-modulecache /tmp/luci-indexcache
```

16G 卡要「变大」：扩分区 / **extroot**，不是软件包页上的 291MB 自己变大。  
长期建议：extroot 或控制日志与节点数量。

---

## 4. Tailscale Exit Node

### 安装与登录

```sh
apk add tailscale
/etc/init.d/tailscale enable
/etc/init.d/tailscale start

# 推荐 Auth Key（一行、密钥无空格）
tailscale up --authkey='tskey-auth-xxxx' \
  --advertise-exit-node \
  --advertise-routes=192.168.124.0/24 \
  --accept-dns=false
```

官网 [Machines](https://login.tailscale.com/admin/machines) **批准**：

- Exit Node
- Subnet `192.168.124.0/24`

### 系统转发

```sh
sysctl -w net.ipv4.ip_forward=1
```

### 测试方法（很重要）

- 手机 **关掉家里 WiFi，用流量**，再选 Exit = 这台 R2S  
- 同一 WiFi 下测 Exit 容易怪异失败  
- 先测 `http://1.1.1.1`（不依赖 DNS），再测域名  
- Windows 客户端若显示 `Tailscale is stopped` / offline，先打开客户端

### SSH 主机密钥变更

重装系统后：

```powershell
ssh-keygen -R 192.168.124.67
ssh root@192.168.124.67
```

---

## 5. Nikki + Tailscale 共存

参考思路（[Cerarin 博文](https://blog.cerarin.com/archives/121)）：

### A. 先让 Exit 稳定（优先）

```sh
# 先停 Nikki 测 Exit
/etc/init.d/nikki stop

# 让 Tailscale 自己管 netfilter（接近「刚装好」）
tailscale set --netfilter-mode=on
tailscale up --reset \
  --advertise-exit-node \
  --advertise-routes=192.168.124.0/24 \
  --accept-dns=false \
  --netfilter-mode=on
```

### B. 需要与 Nikki 精细共存时

1. `tailscale set --netfilter-mode=off`（之后必须自己管防火墙）
2. OpenWrt 防火墙区域 **`tailscale`**：
   - **涵盖的网络** = `tailscale`（绑 `tailscale0` 的接口），**不能是 lan**
   - 入站/出站/转发 = 接受
   - **IP 动态伪装** = 勾选
   - 允许转发到 **lan**
3. 可选：Nikki「局域网代理」入站接口加上 `tailscale`，让异地流量也走分流

手动伪装示例（`netfilter-mode=off` 且计数为 0 时）：

```sh
iptables -I FORWARD -i tailscale0 -j ACCEPT
iptables -I FORWARD -o tailscale0 -j ACCEPT
iptables -t nat -A POSTROUTING -s 100.64.0.0/10 -o br-lan -j MASQUERADE
```

看规则是否吃到包：

```sh
iptables -t nat -L POSTROUTING -n -v
# MASQUERADE 的 pkts 应在手机访问后增加
```

### 路径逻辑

```text
异地设备 → Tailscale → tailscale0
              →（可选）Nikki 分流
              → br-lan → 主路由 → 互联网
```

R2S **本机** Nikki 正常，**不能**推出 Exit 一定正常；Exit 依赖转发 + 伪装。

---

## 6. 常见故障速查

| 现象 | 常见原因 |
|------|----------|
| `ping 223.5.5.5` Network unreachable | 无 default via 网关 |
| `wget Operation not permitted` / apk 源失败 | 同上或 DNS |
| Nikki Not Running | 订阅拉取失败、缺依赖、磁盘满 |
| Tailscale Logged out / NoState | 未登录、中途 Ctrl+C、磁盘满、刚 restart |
| Exit：ping 通、网页超时 | NAT 未命中、同 WiFi 测 Exit、MTU/MSS、DNS |
| iptables POSTROUTING 全 0 包 | netfilter-off 后规则未挂上或流量没进该链 |
| SSH Host key verification failed | 本机 `ssh-keygen -R IP` |
| 软件包页只剩 2MB | overlay 写满，清缓存或 extroot |
| 1000 节点易宕机 | 节点压到 ≤256，少开重服务 |

---

## 7. 防泄露（通用要点）

本仓库**不保存**个人订阅 yaml。通用方向：

1. **DNS**：代理开启时 DNS 也走代理（fake-ip / redir-host + 可信 DNS）；避免系统 DNS 仍指运营商。  
2. **规则**：`GEOIP,CN,DIRECT` 与最终 `MATCH` 策略一致；测试站域名不要误 DIRECT。  
3. **TUN / 系统代理**：确保测浏览器的流量确实进 Nikki，而不是直连。  
4. **Tailscale**：`100.64.0.0/10`、局域网建议 DIRECT，避免环路。  
5. **IPv6**：若未代理 v6，可关 IPv6 或补 v6 规则，减少「双栈泄露」。  
6. 用 [ipleak.net](https://ipleak.net) / browserleaks 分别看 WebRTC、DNS、出口 IP。

具体字段以你当前 Nikki Mixin / 配置文件为准，按面板逐步改即可。

---

## 8. 相关链接

- [OpenWrt Tailscale Wiki](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- [Nikki / OpenWrt-nikki](https://github.com/nikkinikki-org/OpenWrt-nikki)
- [Nikki 与 Tailscale 防火墙共存简述](https://blog.cerarin.com/archives/121)
- [extroot](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration)
- 同账号相关： [Kyousbox](https://github.com/RyuKyou/Kyousbox)（早期 Docker 实验）、[calendar-update-system](https://github.com/RyuKyou/calendar-update-system)

---

## 时间线备忘（本次实验）

1. R2S 刷精简 ImmortalWrt，LAN 当普通设备  
2. 无默认路由 → 无法 apk → 修好 gateway  
3. Nikki 离线 apk + 依赖  
4. Tailscale 登录 / Auth Key / Exit 批准  
5. overlay 99% → 清理与扩容意识  
6. Exit：同网测失败、MASQUERADE 计数为 0、netfilter-mode 拉扯  
7. 停 Nikki + `netfilter-mode=on` + reset 拉起 → **Exit 打通**  
8. 魔法可用；防泄露留给配置细调  

---

## License

文档以记录与分享为目的，按 MIT 许可使用本文中的命令与说明。  
固件、Nikki、Tailscale 等分别遵循各自上游许可。

---

*一次不错的尝试之旅。稳定优先。*
