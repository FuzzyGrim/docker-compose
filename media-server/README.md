# Media Server

## Docker compose of Jellyfin, Sonarr, Radarr, Ombi and Bazarr
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

  ombi:
    image: linuxserver/ombi
    container_name: ombi
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./ombi:/config
    ports:
      - 3579:3579
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
      - ./bazarr_config:/config
      - /data/media/movies:/movies #optional
      - /data/media/tvshows:/tvshows #optional
      - /data/media/anim:/anime
    ports:
      - 6767:6767
    restart: unless-stopped
```
## Docker compose of Prowlarr and Transmission

```yml
version: "2.1"
services:
  prowlarr:
    image: linuxserver/prowlarr:develop
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  transmission:
    image: linuxserver/transmission
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - ./transmission:/config
      - /data/transmission/downloads:/downloads
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    restart: unless-stopped
```