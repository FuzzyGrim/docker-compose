# Media Server

## Docker compose of Jellyfin, Sonarr, Radarr, Jellyseer and Bazarr

```yml
version: "3"
services:
  jellyfin:
    image: linuxserver/jellyfin
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./jellyfin:/config
      - /data/media/tvshows:/data/tvshows
      - /data/media/movies:/data/movies
      - /data/media/anime:/data/anime
    devices:
      - "/dev/dri:/dev/dri"
    ports:
      - 8096:8096
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./sonarr:/config
      - /data/media/anime:/anime
      - /data/media/tvshows:/tvshows
      - /data/transmission/downloads/complete:/downloads/complete
    ports:
      - 8989:8989
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./radarr:/config
      - /data/transmission/downloads/complete:/downloads/complete
      - /data/media/movies:/movies
    ports:
      - 7878:7878
    restart: unless-stopped

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    user: 1000:1000
    environment:
      - TZ=Europe/Madrid
    ports:
      - 5055:5055
    volumes:
      - ./jellyseer_config:/app/config
    restart: unless-stopped
    depends_on:
      - radarr
      - sonarr
  
  bazarr:
    image: linuxserver/bazarr
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./bazarr:/config
      - /data/media/movies:/movies #optional
      - /data/media/tvshows:/tvshows #optional
      - /data/media/anime:/anime
    ports:
      - 6767:6767
    restart: unless-stopped
```

## Docker compose of Prowlarr and Transmission

```yml
version: "3"
services:
  gluetun:
    image: qmcgaw/gluetun
    cap_add:
      - NET_ADMIN
    environment:
      - VPN_SERVICE_PROVIDER=windscribe
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=${PRIVATE_KEY}
      - WIREGUARD_ADDRESSES=${ADDRESSES}
      - WIREGUARD_PRESHARED_KEY=${PRESHARED_KEY}
      - SERVER_REGIONS=Spain
      - SERVER_CITIES=Madrid
      - FIREWALL_OUTBOUND_SUBNETS=192.168.1.0/24
    ports:
      - 9696:9696 # prowlarr
      - 9091:9091 # transmission
      - 51413:51413 # transmission
      - 51413:51413/udp # transmission
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    network_mode: "service:gluetun"
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./prowlarr:/config
    restart: unless-stopped

  transmission:
    image: lscr.io/linuxserver/transmission:latest
    network_mode: "service:gluetun"
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./transmission/config:/config
      - /data/transmission/downloads:/downloads
      - ./transmission/watch:/watch
    restart: unless-stopped
```
