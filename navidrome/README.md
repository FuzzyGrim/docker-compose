# Navidrome

Remember to create the folder /data/navidrome first with user 1000.

## Docker Compose

```yml
version: "3"
services:
  navidrome:
    container_name: navidrome
    user: 1000:1000
    image: deluan/navidrome:latest
    restart: unless-stopped
    networks:
      - proxy
    volumes:
      - "/data/navidrome:/data"
      - "/data/nextcloud/admin/files/music:/music:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.navidrome.entrypoints=https"
      - "traefik.http.routers.navidrome.rule=Host(`navidrome.domain.com`)"
      - "traefik.http.routers.navidrome.tls=true"
    entrypoint: /bin/sh -c "/app/navidrome &> /data/navidrome.log"

networks:
  proxy:
    external: true
```
