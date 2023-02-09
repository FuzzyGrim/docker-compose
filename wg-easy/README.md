# WireGuard Easy

```yml
version: "3.8"
services:
  wg-easy:
    environment:
      - WG_HOST=domain.duckdns.org
      - WG_PORT=4500

    image: weejewel/wg-easy
    container_name: wg-easy
    volumes:
      - ./config:/etc/wireguard
    ports:
      - "4500:51820/udp"
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

```bash
lxc.cap.drop:
```
