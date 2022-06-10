# Uptime Kuma

```yml
version: "3.3"

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    networks:
      - proxy
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.uptime.entrypoints=https"
      - "traefik.http.routers.uptime.rule=Host(`uptime.domain.com`)"
      - "traefik.http.routers.uptime.tls=true"
      - "traefik.http.services.uptime.loadbalancer.server.port=3001"

networks:
  proxy:
    external: true
```
