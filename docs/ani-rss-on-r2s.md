# ani-rss 在 NanoPi R2S / iStoreOS 上的实战笔记

基于 RSS 的自动追番（订阅 → 下载器 → 重命名/刮削）。官方：[wushuo894/ani-rss](https://github.com/wushuo894/ani-rss) · 文档：[docs.wushuo.top](https://docs.wushuo.top)

---

## 硬件与资源提醒

| 项 | 建议 |
|----|------|
| 内存 | R2S 仅 1GB：优先 `openj9` 镜像，`-Xmx192m` 左右；慎本机再上 qB |
| 磁盘 | 配置与媒体放数据分区（如 `/mnt/KyousNAS`），勿挤系统 overlay |
| 角色 | R2S 可作轻量调度；重下载更稳在 NAS/电脑 |

官方更推荐 **qBittorrent**；Aria2 功能支持较少，但更省资源。

---

## Docker 最小示例（host 网络）

```sh
mkdir -p /mnt/KyousNAS/ani-rss/config /mnt/KyousNAS/Media

docker run -d \
  --name ani-rss \
  --restart unless-stopped \
  --network host \
  -e PUID=0 -e PGID=0 -e UMASK=022 \
  -e SERVER_PORT=7789 \
  -e CONFIG=/config \
  -e TZ=Asia/Shanghai \
  -e JAVA_OPTS="-Xms32m -Xmx192m -Xss256k -XX:+UseG1GC" \
  -v /mnt/KyousNAS/ani-rss/config:/config \
  -v /mnt/KyousNAS/Media:/Media \
  wushuo894/ani-rss:openj9
```

面板：`http://R2S的IP:7789`（默认 admin/admin，立刻改密）

---

## 关键：路径必须和下载器一致

官方强调：Docker 下 **ani-rss 与下载器看到的保存路径必须一致**。

### 本机踩坑结论

1. 仅做 `/Media → /mnt/KyousNAS/Media` 软链不够时，常出现 Aria2 `无法建立指定目录 / Permission denied`。
2. **有效做法：让 ani-rss 的「保存位置」与 Aria2 实际下载目录写成同一套路径**（两边都用 Aria2 已能写的目录，例如 Aria2 默认的 `/mnt/KyousNAS/download` 或统一后的 `/Media/...`）。
3. 改路径后需 **保存** 再测试；旧失败任务建议删除后重推。

### 推荐对齐方式（任选一种坚持到底）

**方案 A – 统一到 `/Media`（软链 + 权限）**

```sh
mkdir -p /mnt/KyousNAS/Media/番剧 /mnt/KyousNAS/Media/剧场版
ln -sfn /mnt/KyousNAS/Media /Media
chmod -R a+rwx /mnt/KyousNAS/Media
```

- ani-rss 保存位置：`/Media/番剧/${title}/Season ${season}`
- Aria2 默认目录：`/Media` 或 `/Media/番剧`

**方案 B – 直接改成 Aria2 已有可写路径（本机最终采用的思路）**

- 在 AriaNg 看清「能成功下载」的路径
- ani-rss 下载设置里的保存位置 **改成同一前缀**
- 不再混用未授权目录

---

## Aria2 联调

### 地址与密钥

| 项 | 正确示例 | 错误示例 |
|----|----------|----------|
| 地址 | `http://127.0.0.1:6800/jsonrpc` 或 `http://192.168.124.67:6800/jsonrpc` | 文档站 URL、`http://ttps://docs.wushuo.top/...` |
| 密钥 | 与 AriaNg 相同明文 | 多写 `token:`、系统 root 密码 |

宿主机验证：

```sh
curl -s -X POST http://127.0.0.1:6800/jsonrpc \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":"1","method":"aria2.getVersion","params":["token:你的密钥"]}'
```

ani-rss 使用 **host 网络** 时用 `127.0.0.1` 最省事。

### 日志里曾出现的低级错误

```text
url: http://ttps://docs.wushuo.top/config/download/jsonrpc
```

= 把官方文档链接填进了下载器地址；**改完必须点保存再测试**。

---

## 代理（国内访问蜜柑等）

SSL 握手失败 / DNS 错误时，按官方 FAQ：走代理。

| 项 | 建议 |
|----|------|
| IP | `127.0.0.1`（不要带 `http://`） |
| 端口 | Nikki 混合端口（如 7890） |
| 用户名密码 | Nikki 未开鉴权则留空；开了则与 Nikki 一致 |

验证：

```sh
curl -x http://127.0.0.1:7890 -I --connect-timeout 15 https://mikanime.tv
```

- `407` = 需要代理认证  
- `mikanani.me` 常空响应/TLS 失败，**优先用 mikanime.tv**  
- Nikki 可加：`DOMAIN-SUFFIX,mikanime.tv` / `mikanani.me` / `anibt.net` → 代理组  

---

## Docker 迁移注意

数据目录迁到 KyousNAS 后若出现：

- `Cannot connect to the Docker daemon`
- 首页 Docker「未启用」
- 文件管理 500 / 空目录  

先保证 **`/mnt/KyousNAS` 已挂载**，再启动 dockerd，最后启容器。顺序：挂盘 → Docker → 业务。

---

## 流程清单

1. [ ] 数据盘挂载 + 媒体目录可写  
2. [ ] ani-rss host 网络 + 低内存  
3. [ ] Aria2 RPC 测试成功（地址不是文档链接）  
4. [ ] **保存路径与 Aria2 完全一致**  
5. [ ] 代理指向 Nikki，蜜柑用 mikanime.tv  
6. [ ] 单部测试订阅跑通再批量  

---

## 相关链接

- [下载设置](https://docs.wushuo.top/config/download)  
- [Docker 部署](https://docs.wushuo.top/deploy/docker)  
- [常见问题](https://docs.wushuo.top/faq)  
