# R2S 折腾精华笔记（收尾）

硬件：NanoPi R2S · 系统：iStoreOS / OpenWrt 系 · 角色：LAN 普通设备 + Tailscale Exit + Nikki 分流

---

## 1. 网络角色

- R2S **不要强行当主路由**也可：LAN 静态 IP、关 DHCP，接主路由 LAN。
- 目标路径：

```text
异地设备 → Tailscale Exit → R2S →（可选 Nikki 分流）→ 主路由上网
```

- 对外若走家宽 Exit，检测站显示 **大陆 IP 是正常的**（像在家上网）。
- 比本机 Clash TUN 直连外部代理：在「对外画像」上通常 **更隐蔽**（先回家再出网）。

---

## 2. Tailscale

### 持久 Exit（重启不丢）

```sh
# /etc/tailscale-up.sh
sleep 12
tailscale up --accept-dns=false --advertise-exit-node --advertise-routes=192.168.124.0/24
# 写入 rc.local，并 sysctl net.ipv4.ip_forward=1
```

- 控制台必须 **批准 Use as exit node**。
- **务必 `--accept-dns=false`**，避免和 Nikki/系统 DNS 互抢。

### 临时关闭（测局域网）

```sh
/etc/init.d/tailscale stop
# 恢复：/etc/init.d/tailscale start && /etc/tailscale-up.sh
```

### 与局域网 Samba 慢的关系（实测）

| 状态 | SMB 约速 |
|------|----------|
| Tailscale 运行 | ~300 KB/s（异常） |
| 停止 Tailscale | ~5 MB/s（小文件集合仍受 TF 限制） |

日常不必每次 stop：

- 拷文件只用 **`\\192.168.124.67`**，不要用 `100.x`
- 家里 PC：不要对本机开 Exit；可 `accept-routes=false` 避免把局域网塞进隧道
- 大量小文件：先打 zip 再传

---

## 3. Nikki + Tailscale

| 项 | 建议 |
|----|------|
| 启动顺序 | 网络 → Tailscale → 延迟 20～30s → Nikki |
| DNS 模式 | **Redir-Host**；监听可用 1053，少抢 53 |
| DNS 劫持 | Exit 场景 **先关** |
| TUN | 可开；不稳改 system 栈 / 排除 100.x；时好时坏多因与 TS 抢路由 |
| 混合端口 7890 | 本机/手动代理 |
| 整机与 Exit 一致 | 需透明代理吃到 **转发**，不是只开「允许局域网」 |

---

## 4. 磁盘与挂载

- 数据分区设备：`/dev/mmcblk0p4`
- 挂载点名可以是 `/mnt/mmc0-4` 或 `/mnt/KyousNAS`；**同一分区挂两次则是同一份数据，无需互拷**
- 业务路径统一用 **`/mnt/KyousNAS`**（fstab 改挂载点或软链）
- TF 卡随机写是局域网「只有几 MB/s」的常见上限

---

## 5. ani-rss / 下载

- **保存路径必须与 Aria2 实际可写目录一致**（比只做软链更关键）
- RPC 地址不要填成文档站 URL；改完要 **保存再测**
- 蜜柑优先 **mikanime.tv**；代理走 `127.0.0.1:混合端口`
- 内存紧：停 ani-rss → `docker update --restart=no ani-rss && docker stop ani-rss`

详见 [ani-rss-on-r2s.md](ani-rss-on-r2s.md)

---

## 6. Kavita

- Docker 镜像：`jvmilazz0/kavita` 或 LSIO；R2S 拉 Hub 常 EOF → 电脑 arm64 `docker save` 再 `load`
- 格式：漫画 CBZ/CBR 等；小说 EPUB/PDF
- 远程分享优先 **Tailscale**，勿裸奔 5000 到公网

---

## 7. DNS 抽风（Bing 解析失败等）

- `ERR_NAME_NOT_RESOLVED` = DNS 问题
- 电脑 DNS 尽量用主路由或 `223.5.5.5`
- 关 Tailscale MagicDNS；Nikki 少劫持 53
- 关家用 DDNS **不影响** 局域网 SMB，只影响外网域名回连

---

## 8. 系统清理

见 [istoreos-cleanup.md](istoreos-cleanup.md)

---

## 9. 稳定优先清单

1. [ ] Tailscale：`accept-dns=false` + 开机 `advertise-exit-node` + 控制台批准  
2. [ ] Nikki：Redir-Host、延迟启动、规则清晰  
3. [ ] 数据在 KyousNAS，overlay 不塞业务  
4. [ ] 局域网拷文件走 192.168.x，不走 100.x  
5. [ ] 重服务（ani-rss/Kavita）可停则停，保内存  

*稳定优先。路径对齐一次，少开一天会。*
