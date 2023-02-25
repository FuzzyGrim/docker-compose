# Duplicati

```yml
version: "2.1"
services:
  duplicati:
    image: lscr.io/linuxserver/duplicati
    container_name: duplicati
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - /data/duplicati:/config
      - /backup:/backups
      - /data:/source
    ports:
      - 8200:8200
    restart: unless-stopped
```