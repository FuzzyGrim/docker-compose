# changedetection.io

```
version: '3.3'
services:
  changedetection.io:
    restart: always
    ports:
      - 5000:5000
    volumes:
      - /data/changedetection:/datastore
    container_name: changedetection
    image: dgtlmoon/changedetection.io
    environment:
      - PORT=5000
      - PUID=1000
      - PGID=1000
      - PLAYWRIGHT_DRIVER_URL=ws://playwright-chrome:3000/

  playwright-chrome:
    hostname: playwright-chrome
    image: browserless/chrome
    restart: unless-stopped
```
