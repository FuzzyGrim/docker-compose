# Paperless-ngx

## Docker-compose

```yml
version: "3.4"
services:
  broker:
    container_name: paperless_broker
    image: docker.io/library/redis:7
    restart: unless-stopped
    volumes:
      - /data/paperless/redis:/data

  db:
    container_name: paperless_db
    image: docker.io/library/postgres:13
    restart: unless-stopped
    volumes:
      - /data/paperless/db:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: paperless
      POSTGRES_USER: paperless
      POSTGRES_PASSWORD: ${PW}

  webserver:
    container_name: paperless_webserver
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    restart: unless-stopped
    depends_on:
      - db
      - broker
      - gotenberg
      - tika
    ports:
      - "8010:8000"
    healthcheck:
      test: ["CMD", "curl", "-fs", "-S", "--max-time", "2", "http://localhost:8000"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - /data/paperless/data:/usr/src/paperless/data
      - /data/nextcloud/admin/files/documents:/usr/src/paperless/media
      - /data/paperless/export:/usr/src/paperless/export
      - /data/paperless/consume:/usr/src/paperless/consume
    env_file: docker-compose.env
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBHOST: db
      PAPERLESS_DBPASS: ${PW}
      PAPERLESS_TIKA_ENABLED: 1
      PAPERLESS_TIKA_GOTENBERG_ENDPOINT: http://gotenberg:3000
      PAPERLESS_TIKA_ENDPOINT: http://tika:9998

  gotenberg:
    container_name: paperless_gotenberg
    image: docker.io/gotenberg/gotenberg:7.8
    restart: unless-stopped

    # The gotenberg chromium route is used to convert .eml files. We do not
    # want to allow external content like tracking pixels or even javascript.
    command:
      - "gotenberg"
      - "--chromium-disable-javascript=true"
      - "--chromium-allow-list=file:///tmp/.*"

  tika:
    container_name: paperless_tika
    image: ghcr.io/paperless-ngx/tika:latest
    restart: unless-stopped
```

## Backup

`0 15 * * * docker exec paperless_db pg_dump -h localhost -U paperless paperless > /data/paperless/backups/paperless_$(date '+\%Y-\%m-\%d').sql`
