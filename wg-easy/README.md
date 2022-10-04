# WireGuard Easy

```yml
version: "3.8"
services:
  wg-easy:
    environment:
      - WG_HOST=subdomain.duckdns.org

    image: weejewel/wg-easy
    container_name: wg-easy
    volumes:
      - .:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
```

If you are using Proxmox's LXC, edit `/etc/pve/lxc/<id>.conf` and add:

```
lxc.cap.drop:
```
