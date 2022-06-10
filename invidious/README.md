# Invidious

## Docker Compose
```yml
version: "3"
services:

  invidious:
    image: quay.io/invidious/invidious:latest
    restart: unless-stopped
    networks:
      - default
      - proxy
    environment:
      INVIDIOUS_CONFIG: |
        db:
          dbname: invidious
          user: kemal
          password: ${PASS}
          host: invidious-db
          port: 5432
        check_tables: true
        https_only: true 
        domain: ${URL}
        external_port: 443
        registration_enabled: false
        admins:
          - invidious
    healthcheck:
      test: wget -nv --tries=1 --spider http://127.0.0.1:3000/api/v1/comments/jNQXAC9IVRw || exit 1
      interval: 30s
      timeout: 5s
      retries: 2
    depends_on:
      - invidious-db
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.invidious.entrypoints=https"
      - "traefik.http.routers.invidious.rule=Host(`invidious.domain.com`)"
      - "traefik.http.routers.invidious.tls=true"
      - "traefik.http.services.invidious.loadbalancer.server.port=3000"
      - "traefik.http.routers.invidious-secure.middlewares=authelia@docker"

  invidious-db:
    image: docker.io/library/postgres:14
    restart: unless-stopped
    networks:
      - default
    volumes:
      - postgresdata:/var/lib/postgresql/data
      - ./config/sql:/config/sql
      - ./docker/init-invidious-db.sh:/docker-entrypoint-initdb.d/init-invidious-db.sh
    environment:
      POSTGRES_DB: invidious
      POSTGRES_USER: kemal
      POSTGRES_PASSWORD: ${PASS}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]

volumes:
  postgresdata:

networks:
  proxy:
    external: true
```

## Maintenance
```
docker exec invidious_postgres_1 psql -U kemal invidious -c "DELETE FROM nonces * WHERE expire < current_timestamp"
```
```
docker exec invidious_postgres_1 psql -U kemal invidious -c "TRUNCATE TABLE videos"
```

For regular maintenance you should add a cronjob for these commands:
```
@weekly docker exec invidious_postgres_1 psql -U kemal invidious -c "DELETE FROM nonces * WHERE expire < current_timestamp" > /dev/null

@weekly docker exec invidious_postgres_1 psql -U kemal invidious -c "DELETE FROM nonces * WHERE expire < current_timestamp" > /dev/null
```
