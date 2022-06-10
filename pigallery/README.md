# Pigallery2

```yml
version: '3'
services:

  pigallery2:
    image: bpatrik/pigallery2:latest
    container_name: pigallery2
    networks:
      - proxy
    environment:
      - NODE_ENV=production
    volumes:
      - "./data/config:/app/data/config"
      - "db-data:/app/data/db"
      - "./data/images:/app/data/images:ro"
      - "./data/tmp:/app/data/tmp"
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.pigallery.entrypoints=https"
      - "traefik.http.routers.pigallery.rule=Host(`pigallery.domain.com`)"
      - "traefik.http.routers.pigallery.tls=true"
      - "traefik.http.services.pigallery.loadbalancer.server.port=80"

volumes:
  db-data:

networks:
  proxy:
    external: true
```

## Config
To allow [public browsing](https://github.com/bpatrik/pigallery2/issues/60) :
 - Disable password protection from GUI.
 - Manually edit config.json and set unAuthenticatedUserRole to a different value (Admin to User).