# Caddy

## Prerequisites

Get custom [Caddy build](https://caddyserver.com/download) by selecting cloudflare and download. Make sure the caddy file is executable (`chmod a+x caddy`).

#### A record config

Create an API token for the DNS challenge:

1. In the upper right, click the person icon and navigate to My Profile, and then select the API Tokens tab.
2. Click the Create Token button, and then Use template on Edit zone DNS.
3. Edit the Token name field if you prefer a more descriptive name.
4. Under Permissions, the Zone / DNS / Edit permission should already be populated. Add another permission: Zone / Zone / Read.
5. Under Zone Resources, set Include / Specific zone / example.com (replacing example.com with your domain).
6. Under TTL, set an End Date for when your token will become inactive. You might want to choose one far  in the future.
7. Create the token and copy the token value.

## Docker Compose
```yml
version: '3'

services:
  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # Your custom build of Caddy.
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - EMAIL=${email}
      - CLOUDFLARE_API_TOKEN=${cloudflare_token}
      - LOG_FILE=/data/access.log
```

## Caddyfile in same path as docker-compose.yml 

```
domain:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # Use the ACME DNS-01 challenge to get a cert for the configured domain.
  tls {
    dns cloudflare {$CLOUDFLARE_API_TOKEN}
  }

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  encode gzip

  # Notifications redirected to the WebSocket server
  reverse_proxy /notifications/hub 192.168.1.126:3012

  # Proxy everything else to Rocket
  reverse_proxy 192.168.1.126:8086
}

domain:443 {
  # Use the ACME DNS-01 challenge to get a cert for the configured domain.
  tls {
    dns cloudflare {$CLOUDFLARE_API_TOKEN}
  }
  
  reverse_proxy 192.168.1.126:6875
}
```