# Epic Games Store Weekly Free Games

```yml
version: '3.3'
services:
  epicgames-freegames:
    container_name: epicgames-freegames
    environment:
      - TZ=Europe/Madrid
      - EMAIL=${EMAIL}
      - PASSWORD=${PASSWORD}
      - DISCORD_WEBHOOK=${DISCORD_WEBHOOK}            
      - RUN_ON_STARTUP=true
      - BASE_URL=http://192.168.1.122:3001
    volumes:
      - './data:/usr/app/config:rw'
    ports:
      - '3001:3000'
    image: 'charlocharlie/epicgames-freegames:latest'
    restart: unless-stopped
```
