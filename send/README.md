# Send

## Docker-compose

```yml
version: "3"

services:
  send:
    image: 'registry.gitlab.com/timvisee/send:latest'
    container_name: send
    restart: always
    networks:
      - proxy
      - default
    volumes:
      - ./uploads:/uploads
    environment:
      - VIRTUAL_HOST=send.domain.com
      - VIRTUAL_PORT=1234
      - DHPARAM_GENERATION=false
      - NODE_ENV=production
      - BASE_URL=https://send.domain.com
      - PORT=1234
      - REDIS_HOST=redis

      # To customize upload limits
      # - EXPIRE_TIMES_SECONDS=3600,86400,604800,2592000,31536000
      # - DEFAULT_EXPIRE_SECONDS=3600
      # - MAX_EXPIRE_SECONDS=31536000
      # - DOWNLOAD_COUNTS=1,2,5,10,15,25,50,100,1000
      # - MAX_DOWNLOADS=1000
      - MAX_FILE_SIZE=5368709120

    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.send.entrypoints=https"
      - "traefik.http.routers.send.rule=Host(`send.domain.com`)"
      - "traefik.http.routers.send.tls=true"
      - "traefik.http.services.send.loadbalancer.server.port=1234"
      - "traefik.http.routers.send.middlewares=authelia@docker"

  redis:
    container_name: send-redis
    networks:
      - default
    image: 'redis:alpine'
    restart: always
    volumes:
      - send-redis:/data

volumes:
  send-redis:

networks:
  proxy:
    external: true
```