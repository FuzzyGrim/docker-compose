# Code Server

## Docker Compose
```yml
version: "2.1"
services:
  code-server:
    image: lscr.io/linuxserver/code-server
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - PASSWORD=${PASS}
      - SUDO_PASSWORD=${PASS}
      - PROXY_DOMAIN=${URL} #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - /data/vscode/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```