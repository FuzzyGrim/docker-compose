# Vaultwarden

## Docker Compose
```yml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # Enable WebSocket notifications.
    volumes:
      - ./vw-data:/data
    environment:
      - SIGNUPS_ALLOWED=false
      - IP_HEADER=CF-Connecting-IP
      - LOG_FILE=/data/vaultwarden.log
      - LOG_LEVEL=warn
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.vaultwarden.entrypoints=https"
      - "traefik.http.routers.vaultwarden.rule=Host(`vaultwarden.domain.com`)"
      - "traefik.http.routers.vaultwarden.tls=true"
      - "traefik.http.services.vaultwarden.loadbalancer.server.port=80"

networks:
  proxy:
    external: true
```

## Start
`docker-compose up -d`

## Backup
Add to `crontab -e`
```
sqlite3 /data/vaultwarden/db.sqlite3 ".backup '/data/vaultwarden/db_bak/db-$(date '+\%Y-\%m-\%d').sqlite3'"
```
