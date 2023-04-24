# Umami

First copy [schema](https://github.com/mikecao/umami/blob/master/sql/schema.postgresql.sql)
to `sql/schema.postgresql.sql`.

```yml
version: '3'
services:
  umami:
    container_name: umami
    image: ghcr.io/mikecao/umami:postgresql-latest
    networks:
      - proxy
      - default
    environment:
      - DATABASE_URL=postgresql://umami:${PASS}@db:5432/umami
      - DATABASE_TYPE=postgresql
      - HASH_SALT=${SALT}
      - REMOVE_TRAILING_SLASH=1
      - TRACKER_SCRIPT_NAME=${SCRIPT}
    depends_on:
      - db
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.umami.entrypoints=https"
      - "traefik.http.routers.umami.rule=Host(`umami.domain.com`)"
      - "traefik.http.routers.umami.tls=true"
      - "traefik.http.services.umami.loadbalancer.server.port=3000"

  db:
    container_name: umami-db
    image: postgres:12-alpine
    networks:
      - default
    environment:
      - POSTGRES_DB=umami
      - POSTGRES_USER=umami
      - POSTGRES_PASSWORD=${PASS}
    volumes:
      - ./sql/schema.postgresql.sql:/docker-entrypoint-initdb.d/schema.postgresql.sql:ro
      - ./db:/var/lib/postgresql/data
    restart: unless-stopped

networks:
  proxy:
    external: true
```

## Backup

`0 21 * * * docker exec umami-db pg_dump -h localhost -U umami umami > /home/user/docker/umami/backups/umami_$(date '+\%Y-\%m-\%d').sql`
