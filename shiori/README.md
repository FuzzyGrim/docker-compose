# Shiori

```yml
version: "3"
services:
  shiori:
    image: radhifadlillah/shiori:latest
    container_name: shiori
    volumes:
      - ./:/srv/shiori
    restart: unless-stopped

    networks:
      - proxy

    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.shiori.entrypoints=https"
      - "traefik.http.routers.shiori.rule=Host(`shiori.domain.com`)"
      - "traefik.http.routers.shiori.tls=true"
      - "traefik.http.services.shiori.loadbalancer.server.port=8080"

networks:
  proxy:
    external: true
```

## Startup

Since this is our first time, we don't have any account registered yet. With that said, we can use the default user to access web interface :

```
username: shiori
password: gopher
```

The first new account you add will become the owner and it will deactivate the "shiori:gopher" default user automatically.
