# iStoreOS / OpenWrt 缓存与垃圾清理

适用于 NanoPi R2S 等小闪存设备。清理前建议先 `df -h` 看空间。

---

## 安全、可常做

```sh
# 1. 软件包缓存（apk）
apk cache clean 2>/dev/null
rm -rf /var/cache/apk/* 2>/dev/null

# 若仍是 opkg 环境
opkg clean 2>/dev/null
rm -rf /tmp/opkg-lists/* 2>/dev/null

# 2. 临时目录（重启也会清一部分）
rm -rf /tmp/luci-* /tmp/luci-indexcache* 2>/dev/null
# 勿删正在用的关键 socket；一般重启更干净

# 3. 日志（过大时）
logread -c 2>/dev/null
# 或
> /var/log/messages 2>/dev/null
rm -f /tmp/log/* 2>/dev/null

# 4. 看空间
df -h
du -sh /overlay /tmp /mnt/KyousNAS 2>/dev/null
```

---

## Docker 相关（有 Docker 时）

```sh
# 停掉不需要的容器后再清
docker container prune -f
docker image prune -f          # 只删悬空镜像
# 更狠（会删所有未使用镜像，慎用）：
# docker system prune -af

# 日志过大时
truncate -s 0 $(docker inspect --format='{{.LogPath}}' 容器名) 2>/dev/null
```

数据在 `/mnt/KyousNAS` 时，不要乱删该目录下的 `Media`、`config`。

---

## Overlay 紧张时

```sh
df -h /overlay
du -sh /overlay/upper/* 2>/dev/null | sort -hr | head

# 卸掉不用的包
apk del 包名
# 或 opkg remove 包名
```

长期方案：数据与 Docker 根目录放 **数据分区**（如 `/mnt/KyousNAS`），勿挤 overlay。

---

## 不建议

- 删除 `/etc/config/*`、正在跑的服务状态目录（除非知道后果）
- 在业务运行时对数据盘 `rm -rf`
- 把「清理」当成解决网络问题的第一手段（先查 DNS/Nikki/Tailscale）

---

## 一键温和清理示例

```sh
apk cache clean 2>/dev/null
rm -rf /tmp/luci-* /tmp/opkg-lists 2>/dev/null
docker container prune -f 2>/dev/null
docker image prune -f 2>/dev/null
df -h
```
