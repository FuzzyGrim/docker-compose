# Libreddit

## Docker compose
```yml
version: '3'

services:
  libreddit:
    container_name: "libreddit"
    image: spikecodes/libreddit
    user: "1000:1000"
    restart: unless-stopped
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.libreddit.entrypoints=https"
      - "traefik.http.routers.libreddit.rule=Host(`libreddit.domain.com`)"
      - "traefik.http.routers.libreddit.tls=true"
      - "traefik.http.services.libreddit.loadbalancer.server.port=8080"
      - "traefik.http.routers.invidious.middlewares=authelia@docker"

networks:
  proxy:
    external: true
```