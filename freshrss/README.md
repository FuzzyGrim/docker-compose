# FreshRSS

```yml
version: "2.1"
services:
  freshrss:
    image: lscr.io/linuxserver/freshrss
    container_name: freshrss
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - /data/freshrss:/config
    restart: unless-stopped
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.freshrss.entrypoints=https"
      - "traefik.http.routers.freshrss.rule=Host(`freshrss.domain.com`)"
      - "traefik.http.routers.freshrss.tls=true"
      - "traefik.http.services.freshrss.loadbalancer.server.port=80"

networks:
  proxy:
    external: true
```