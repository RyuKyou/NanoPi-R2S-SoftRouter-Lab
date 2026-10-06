# 同类 RSS / 追更工具（视频 · 漫画）

与 ani-rss 类似的自动化订阅方案，便于在 NAS / 软路由 / 电脑上选型。

---

## 动画 / 视频向

| 项目 | 特点 | 备注 |
|------|------|------|
| **[ani-rss](https://github.com/wushuo894/ani-rss)** | RSS 追番、多下载器、重命名/刮削 | 本仓库已有 R2S 笔记；官方偏爱 qB |
| **[AutoBangumi](https://github.com/EstrellaXD/AutoBangumi)** | 蜜柑 RSS + qB，较经典 | 社区方案多，与 ani-rss 同赛道 |
| **[punishing-downloader](https://github.com/Flood-and-Fire/punishing)** 等 | 各有侧重 | 自行甄别维护状态 |
| **Sonarr** | 剧集库管理 + RSS/种子索引 | 偏西方剧集生态，重、要 TMDB 等 |
| **Radarr** | 电影 | 与 Sonarr 一套 |
| **FlexGet** | 通用 RSS/订阅引擎 | 可玩性高，配置陡 |
| **qBit + RSS 自带** | 无额外追番逻辑 | 最轻，无刮削/番剧元数据 |

R2S 1GB：优先 **ani-rss 轻量配置** 或 **AutoBangumi 看内存**；Sonarr 全家桶更适合 NAS。

---

## 漫画向

| 项目 | 特点 | 备注 |
|------|------|------|
| **[LANraragi](https://github.com/Difegue/LANraragi)** | 漫画库 + 插件/下载器生态 | Docker 友好 |
| **[Komga](https://github.com/gotson/komga)** | 漫画/电子书服务器 | 偏「已有文件」管理与阅读 |
| **[Kavita](https://github.com/Kareadita/Kavita)** | 漫画/小说阅读器 | 同上 |
| **[Tachiyomi](https://github.com/tachiyomiorg/tachiyomi) 系 / Mihon** | 手机端追漫 | 不是服务端 RSS 中心 |
| **[Suwayomi-Server](https://github.com/Suwayomi/Suwayomi-Server)**（原 Tachidesk） | 服务端化漫画扩展 | 可挂 NAS，注意源与合规 |
| **FlexGet / 通用 RSS** | 自己对接漫画 RSS | 灵活，要自己写规则 |

漫画更常是「阅读器 + 自行归档」；全自动追漫对源站依赖强，请遵守当地法律与站点条款。

---

## 选型建议（结合 R2S）

```text
只追番、要省内存     → ani-rss + Aria2（路径对齐）或远端 qB
要完整影视库         → 另机 Sonarr/Radarr + qB
漫画阅读为主         → Komga/Kavita 管库；追更用手机端或 Suwayomi
软路由只做跳板       → R2S 跑 Nikki/Tailscale；ani-rss 放电脑/NAS
```

---

## 合规说明

以上工具多为中立技术组件。订阅与下载内容的合法性由使用者自行负责，请遵守著作权与当地法规。
