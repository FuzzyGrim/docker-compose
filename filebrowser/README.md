# FileBrowser

```yml
version: '3.3'
services:
  filebrowser:
    container_name: filebrowser
    volumes:
      - '/:/srv'
      - './database:/database'
      - './config:/config'
    environment:
      - PUID=1000
      - PGID=1000
    ports:
      - '8300:80'
    image: 'filebrowser/filebrowser:s6'
    restart: always
```