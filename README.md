# NanoPi R2S SoftRouter Lab

> NanoPi R2S 上跑 **Nikki (Mihomo)** + **Tailscale Exit Node** + 可选下载/书库的实战笔记。  
> 从刷机、踩坑到收尾的完整路径。

**硬件**：FriendlyELEC NanoPi R2S（RK3328 / 1GB RAM）  
**系统**：iStoreOS / ImmortalWrt 等 OpenWrt 系  
**角色**：主路由 LAN 下的「普通设备」+ 24h 代理保底 + Tailscale 出口

公开仓库，方便回顾与后来者少踩坑。

---

## 文档目录

| 文档 | 内容 |
|------|------|
| [README（本页）](README.md) | 总览 |
| [docs/session-notes-essentials.md](docs/session-notes-essentials.md) | **本轮折腾精华（收尾）** |
| [docs/istoreos-cleanup.md](docs/istoreos-cleanup.md) | 系统缓存/垃圾清理命令 |
| [docs/ani-rss-on-r2s.md](docs/ani-rss-on-r2s.md) | ani-rss + Aria2 路径对齐 |
| [docs/rss-media-alternatives.md](docs/rss-media-alternatives.md) | 追番/漫画/小说类工具 |
| [docs/checklist.md](docs/checklist.md) | 上线检查清单 |
| [docs/commands-cheatsheet.md](docs/commands-cheatsheet.md) | 命令速查 |

---

## 架构一览

```text
主路由
  └─ R2S（静态 LAN IP）
        ├─ Nikki：分流 / 可选 TUN
        ├─ Tailscale：Exit + 可选子网（accept-dns=false）
        └─ 数据盘 /mnt/KyousNAS：Docker、Media、配置
```

### 关键结论（极简）

1. **Exit 持久化**：开机脚本写死 `--advertise-exit-node`，控制台批准。  
2. **DNS**：Tailscale 不要管 DNS；Nikki 用 Redir-Host，少抢 53。  
3. **局域网拷文件**：用 `192.168.x`，勿走 `100.x`；TS 开着时 SMB 可能极慢（实测）。  
4. **ani-rss**：保存路径与 Aria2 **完全一致**。  
5. **挂载**：`mmc0-4` 与 `KyousNAS` 若同为 p4，无需互拷。  
6. **内存**：1GB 上重服务可停；清理见 cleanup 文档。  

---

## 相关仓库

- [Kyousbox](https://github.com/RyuKyou/Kyousbox)  
- [calendar-update-system](https://github.com/RyuKyou/calendar-update-system)  

---

## License

文档以记录与分享为目的。上游软件遵循其各自许可。

*谢谢一路折腾。稳定优先。☕*
