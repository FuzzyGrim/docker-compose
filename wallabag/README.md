# Wallabag

```yml
version: '3.3'
services:
  wallabag:
    container_name: wallabag
    image: wallabag/wallabag
    restart: unless-stopped
    volumes:
      - ./data:/var/www/wallabag/data
      - ./images:/var/www/wallabag/web/assets/images
    networks:
      - proxy
    environment:
      - SYMFONY__ENV__FOSUSER_REGISTRATION=false
      - SYMFONY__ENV__DOMAIN_NAME=https://wallabag.domain.com
      - SYMFONY__ENV__FOSUSER_CONFIRMATION=false
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.wallabag.entrypoints=https"
      - "traefik.http.routers.wallabag.rule=Host(`wallabag.domain.com`)"
      - "traefik.http.routers.wallabag.tls=true"
      - "traefik.http.services.wallabag.loadbalancer.server.port=80"
      - "traefik.http.routers.wallabag.middlewares=authelia@docker"

networks:
  proxy:
    external: true
```