# Home Assistant

## Docker Compose
```yml
version: "2.1"
services:
  homeassistant:
    image: lscr.io/linuxserver/homeassistant
    container_name: homeassistant
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - /data/homeassistant:/config
    restart: unless-stopped
```

# HACS

Use: `wget -O - https://get.hacs.xyz | bash -`