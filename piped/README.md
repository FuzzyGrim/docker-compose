# Piped

First, install `git`, `docker` and `docker-compose`.

Run `git clone https://github.com/TeamPiped/Piped-Docker`.

Then, run `cd Piped-Docker`.

Then, run `./configure-instance.sh` and fill in the hostnames when asked. Choose nginx as the reverse proxy when asked.

The hostnames I have chosen: `piped`, `pipedapi`, `pipedproxy`.

## Docker compose (Change domain and POSTGRES_PASSWORD)

```yml
services:
    pipedfrontend:
        image: 1337kavin/piped-frontend:latest
        restart: unless-stopped
        depends_on:
            - piped
        container_name: piped-frontend
        entrypoint: ash -c 'sed -i s/pipedapi.kavin.rocks/pipedapi.domain.com/g
            /usr/share/nginx/html/assets/* && /docker-entrypoint.sh && nginx -g
            "daemon off;"'
        networks:
            - proxy
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.piped.entrypoints=https"
            - "traefik.http.routers.piped.rule=Host(`piped.domain.com`)"
            - "traefik.http.routers.piped.tls=true"
            - "traefik.http.services.piped.loadbalancer.server.port=80"
            - "traefik.http.routers.piped.middlewares=authelia@docker"

    ytproxy:
        image: 1337kavin/ytproxy:latest
        restart: unless-stopped
        networks:
            - default
        volumes:
            - ytproxy:/app/socket
        container_name: piped-ytproxy

    piped:
        image: 1337kavin/piped:latest
        restart: unless-stopped
        volumes:
            - ./config/config.properties:/app/config.properties:ro
        networks:
            - default
        depends_on:
            - postgres
        container_name: piped-backend

    varnish:
        image: varnish:7.0-alpine
        restart: unless-stopped
        networks:
            - default
            - proxy
        volumes:
            - ./config/default.vcl:/etc/varnish/default.vcl:ro
        container_name: piped-varnish
        depends_on:
            - piped
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.pipedapi.entrypoints=https"
            - "traefik.http.routers.pipedapi.rule=Host(`pipedapi.domain.com`)"
            - "traefik.http.routers.pipedapi.tls=true"
            - "traefik.http.services.pipedapi.loadbalancer.server.port=80"
        healthcheck:
            test: ash -c "wget --no-verbose --tries=1 --spider 127.0.0.1:80/feed ||
                (varnishreload && exit 1)"
            interval: 10s
            timeout: 10s
            retries: 1

    nginx:
        image: nginx:mainline-alpine
        restart: unless-stopped
        networks:
            - proxy
            - default
        volumes:
            - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
            - ./config/pipedapi.conf:/etc/nginx/conf.d/pipedapi.conf:ro
            - ./config/pipedproxy.conf:/etc/nginx/conf.d/pipedproxy.conf:ro
            - ./config/pipedfrontend.conf:/etc/nginx/conf.d/pipedfrontend.conf:ro
            - ./config/ytproxy.conf:/etc/nginx/snippets/ytproxy.conf:ro
            - ytproxy:/var/run/ytproxy
        container_name: piped-nginx
        depends_on:
            - piped
            - varnish
            - ytproxy
            - pipedfrontend
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.pipedproxy.entrypoints=https"
            - "traefik.http.routers.pipedproxy.rule=Host(`pipedproxy.domain.com`)"
            - "traefik.http.routers.pipedproxy.tls=true"
            - "traefik.http.services.pipedproxy.loadbalancer.server.port=80"

    postgres:
        image: postgres:13-alpine
        restart: unless-stopped
        networks:
            - default
        volumes:
            - ./data/db:/var/lib/postgresql/data
        environment:
            - POSTGRES_DB=piped
            - POSTGRES_USER=piped
            - POSTGRES_PASSWORD={DB_PW}
        container_name: piped-postgres

volumes:
    ytproxy: null
networks:
    proxy:
        external: true
```
