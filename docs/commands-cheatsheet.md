# 命令速查

## 网络

```sh
ip route
ip route add default via 192.168.124.1 dev br-lan
ping -c 3 223.5.5.5
uci show network.lan
```

## 磁盘

```sh
df -h
du -sh /overlay/* 2>/dev/null | sort -h | tail
rm -f /tmp/*.apk /tmp/*.tar.gz
```

## Nikki

```sh
apk info | grep -iE 'nikki|mihomo'
/etc/init.d/nikki status
/etc/init.d/nikki restart
logread | grep -iE 'nikki|mihomo' | tail -40
```

## Tailscale

```sh
tailscale status
tailscale ip -4
tailscale set --netfilter-mode=on
tailscale set --netfilter-mode=off
tailscale set --accept-dns=false
tailscale up --reset --advertise-exit-node \
  --advertise-routes=192.168.124.0/24 \
  --accept-dns=false --netfilter-mode=on

sysctl -w net.ipv4.ip_forward=1
iptables -t nat -L POSTROUTING -n -v
iptables -L FORWARD -n -v | head
```

## 防火墙区域（思路）

- 接口：`tailscale` / 设备 `tailscale0` / Unmanaged
- 区域名 `tailscale`：涵盖网络 = **tailscale**（不是 lan）
- masq 开；转发到 lan

## SSH（Windows）

```powershell
ssh-keygen -R 192.168.124.67
ssh root@192.168.124.67
```
