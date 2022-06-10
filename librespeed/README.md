# LibreSpeed

## Docker compose
```yml
version: "2.1"
services:
  librespeed:
    image: linuxserver/librespeed
    container_name: librespeed
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - ./config:/config
    ports:
      - 30001:80
    restart: unless-stopped
```