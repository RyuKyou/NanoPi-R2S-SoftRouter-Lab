# 上线检查清单

## 网络

- [ ] `ip route` 有 `default via <主路由>`
- [ ] `ping 223.5.5.5` 通
- [ ] lan DHCP 已 ignore，无地址冲突

## 磁盘

- [ ] `df -h` overlay 剩余 > 30MB（越好越大）
- [ ] 无长期 99% 占用

## Nikki

- [ ] Core Status = Running
- [ ] 节点数可控（建议 ≤256）
- [ ] 面板能连（优先自带 Dashboard）

## Tailscale

- [ ] `tailscale status` 非 Logged out / NoState
- [ ] 官网 Exit Node 已批准
- [ ] 子网路由已批准（若需要）
- [ ] `net.ipv4.ip_forward=1`
- [ ] 手机用**流量**测 Exit，先 `http://1.1.1.1` 再域名

## 共存

- [ ] 先 Exit 通，再开 Nikki
- [ ] `100.64.0.0/10` 与局域网 DIRECT（避免环路）
- [ ] 若 `netfilter-mode=off`：区域涵盖网络 = `tailscale`，masq 开，且规则有包计数
