# Bookstack

## Docker-Compose

```yml
version: "3"
services:
  bookstack:
    image: lscr.io/linuxserver/bookstack
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - APP_URL=${URL}
      - DB_HOST=bookstack_db
      - DB_USER=bookstack
      - DB_PASS=${DB_PW}
      - DB_DATABASE=bookstackapp
    volumes:
      - /data/bookstack/config:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack_db

  bookstack_db:
    image: lscr.io/linuxserver/mariadb
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=${DB_PW}
      - TZ=Europe/Madrid
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=${DB_PW}
    volumes:
      - /data/bookstack/database/config:/config
    restart: unless-stopped
```