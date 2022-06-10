# Commentoplusplus

```yml
version: '3.7'

services:
  commento:
    container_name: commento
    image: caroga/commentoplusplus:latest
    networks:
      - default
      - proxy
    restart: unless-stopped
    environment:
      - COMMENTO_ORIGIN=https://commento.domain.com
      - COMMENTO_PORT=8080
      - COMMENTO_POSTGRES=postgres://postgres:${PASS}@db:5432/commento?sslmode=disable
      - COMMENTO_GZIP_STATIC=true
      - COMMENTO_FORBID_NEW_OWNERS=true
      - COMMENTO_SMTP_HOST=in-v3.mailjet.com
      - COMMENTO_SMTP_PORT=587
      - COMMENTO_SMTP_USERNAME=${USER}
      - COMMENTO_SMTP_PASSWORD=${PASSWORD}
      - COMMENTO_SMTP_FROM_ADDRESS=mail@domain.com
    depends_on:
      - db
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.commento.entrypoints=https"
      - "traefik.http.routers.commento.rule=Host(`commento.domain.com)"
      - "traefik.http.routers.commento.tls=true"
      - "traefik.http.services.commento.loadbalancer.server.port=8080"

  db:
    container_name: commento-db
    image: postgres:latest
    networks:
      - default
    restart: unless-stopped
    environment:
      - POSTGRES_DB=commento
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${DB_PASS}
    volumes:
      - ./data:/var/lib/postgresql/data

networks:
  proxy:
    external: true
```

## Backup

Run: `docker exec commento-db pg_dump -h localhost -U postgres commento > /home/user/docker/commentoplusplus/backups/commento_$(date '+%Y%m%d').sql`

On crontab:
```
0 3 * * * docker exec commento-db pg_dump -h localhost -U postgres commento > /home/user/docker/commentoplusplus/backups/commento_$(date '+\%Y-\%m-\%d').sql
```