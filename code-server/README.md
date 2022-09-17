# Code Server

## Docker Compose
```yml
version: "2.1"
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - HASHED_PASSWORD=${PASS}
      - SUDO_PASSWORD_HASH=${SUDO}
      - PROXY_DOMAIN=${URL}
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - ./config:/config
    restart: unless-stopped
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.code-server.entrypoints=https"
      - "traefik.http.routers.code-server.rule=Host(`coder.domain.com`)"
      - "traefik.http.routers.code-server.tls=true"
      - "traefik.http.services.code-server.loadbalancer.server.port=8443"

networks:
  proxy:
    external: true
```
