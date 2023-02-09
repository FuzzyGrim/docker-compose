# Minecraft Server

## Minecraft server docker image

```yml
version: "3"

services:
  mc:
    image: itzg/minecraft-server
    container_name: minecraft-server
    ports:
      - 25565:25565
    environment:
      EULA: "TRUE"
      TYPE: "FORGE"
      DIFFICULTY: "normal"
    tty: true
    stdin_open: true
    restart: unless-stopped
    volumes:
      - ./data:/data
```

## Lightweight proxy for Minecraft Java server

```yml
version: '3.4'

services:
  router:
    image: ${MC_ROUTER_IMAGE:-itzg/mc-router}
    environment:
      # enable API
      API_BINDING: ":25564"
    ports:
      - 25565:25565
      # bind the API port to only loopback to avoid external exposure
      - 127.0.0.1:25564:25564
    command: --mapping=minecraft.example.com=192.168.x.x:25565
```
