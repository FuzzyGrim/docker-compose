# Fail2ban

```yml
version: "3.5"

services:
  fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - "./data:/data"
      - "/data/authelia/log:/var/log/authelia:ro"
      - "/data/nextcloud/nextcloud.log:/var/log/nextcloud/nextcloud.log:ro"
      - "/data/vaultwarden/vaultwarden.log:/var/log/vaultwarden/vaultwarden.log:ro"
      - "/data/navidrome/navidrome.log:/var/log/navidrome/navidrome.log:ro"
      - "/var/log/traefik:/var/log/traefik/:ro"
    restart: always
```