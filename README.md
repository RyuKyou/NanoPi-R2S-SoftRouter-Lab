# NanoPi R2S SoftRouter Lab

> NanoPi R2S 上跑 **Nikki (Mihomo)** + **Tailscale Exit Node** + 可选 **ani-rss** 等的实战笔记。  
> 从刷机、踩坑到打通的完整路径。

**硬件**：FriendlyELEC NanoPi R2S（RK3328 / 1GB RAM）  
**系统**：ImmortalWrt / iStoreOS 等 OpenWrt 系  
**角色**：接主路由 LAN 的「普通设备」+ 24h 代理保底 + Tailscale 出口（可选追番调度）

公开仓库，方便回顾与后来者少踩坑。

---

## 文档目录

| 文档 | 内容 |
|------|------|
| [README（本页）](README.md) | 总览、网络、Nikki、Tailscale、磁盘 |
| [docs/checklist.md](docs/checklist.md) | 上线检查清单 |
| [docs/commands-cheatsheet.md](docs/commands-cheatsheet.md) | 命令速查 |
| [docs/ani-rss-on-r2s.md](docs/ani-rss-on-r2s.md) | **ani-rss + Aria2 实战（路径对齐等）** |
| [docs/rss-media-alternatives.md](docs/rss-media-alternatives.md) | 同类视频/漫画 RSS 与库管理工具 |

---

## 目标架构

```text
主路由 (如 192.168.124.1)
    │
    └─ R2S LAN 静态 IP（如 .67）
           ├─ Nikki / Mihomo
           ├─ Tailscale（Exit + 可选子网）
           ├─ 可选：ani-rss + Aria2（数据在 /mnt/KyousNAS）
           └─ 可选：OpenList 等
```

- **不做** 强行旁路由劫持全家（可按需求再开）  
- 节点订阅建议控制数量，1GB 易 OOM  
- overlay 紧张时扩容/extroot 或数据外置  

---

## 1. 网络：普通设备模式

```sh
uci set network.lan.proto='static'
uci set network.lan.ipaddr='192.168.124.67'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='192.168.124.1'
uci set network.lan.dns='192.168.124.1'
uci set dhcp.lan.ignore='1'
uci commit network && uci commit dhcp
/etc/init.d/network restart

ip route   # 需有 default via 主路由
```

---

## 2. Nikki / Tailscale（摘要）

- Nikki：入站/TUN 按「是否只服务 Exit」裁剪，避免误伤局域网  
- Tailscale：`--advertise-exit-node`、子网按需、`ip_forward=1`  
- 异地测 Exit 用 **流量网络**；先通 Exit 再叠加 Nikki  
- 详见早期排查：`netfilter-mode`、MASQUERADE、区域涵盖 `tailscale`  

参考：[Cerarin - Nikki 与 Tailscale](https://blog.cerarin.com/archives/121)

---

## 3. ani-rss 要点（摘要）

完整步骤见 **[docs/ani-rss-on-r2s.md](docs/ani-rss-on-r2s.md)**。

1. Docker `openj9` + host 网络 + 数据在 KyousNAS  
2. Aria2 地址勿填文档链接；改完 **先保存再测试**  
3. **保存路径与 Aria2 下载目录完全一致**（本机最终靠「对齐路径」解决无法建目录）  
4. 代理走 Nikki `127.0.0.1:混合端口`；蜜柑优先 **mikanime.tv**  

---

## 4. 磁盘与 Docker

- 数据盘挂载点示例：`/mnt/KyousNAS`  
- 迁移 Docker 根目录后：先挂盘，再启 dockerd  
- 文件管理 500：检查挂载与根路径配置  

---

## 5. 相关仓库

- [Kyousbox](https://github.com/RyuKyou/Kyousbox)（早期 Docker 实验）  
- [calendar-update-system](https://github.com/RyuKyou/calendar-update-system)  

---

## License

文档以记录与分享为目的。固件与各上游软件遵循其各自许可。

*稳定优先。路径对齐一次，少开一天会。*
