# Duck DNS
```yml
version: "2.1"
services:
  duckdns:
    image: lscr.io/linuxserver/duckdns:latest
    container_name: duckdns
    environment:
      - PUID=1000 #optional
      - PGID=1000 #optional
      - TZ=Europe/Madrid
      - SUBDOMAINS=${SUBDOMAIN}
      - TOKEN=${TOKEN}
      - LOG_FILE=true
    volumes:
      - ./config:/config #optional
    restart: unless-stopped
```
