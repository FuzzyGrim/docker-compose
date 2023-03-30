# Authelia

Check my [guide](https://www.fuzzygrim.com/posts/exposing-services) on how to setup Authelia with Traefik.

## Docker-compose

```yml
version: '3'

services:
  authelia:
    image: authelia/authelia
    container_name: authelia
    volumes:
      - /data/authelia:/config
      - /data/authelia/log:/var/log/authelia
    networks:
      - proxy
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.authelia.rule=Host(`auth.domain.com`)'
      - 'traefik.http.routers.authelia.entrypoints=https'
      - 'traefik.http.routers.authelia.tls=true'
      - 'traefik.http.middlewares.authelia.forwardauth.address=http://authelia:9091/api/verify?rd=https://auth.domain.com'
      - 'traefik.http.middlewares.authelia.forwardauth.trustForwardHeader=true'
      - 'traefik.http.middlewares.authelia.forwardauth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
    expose:
      - 9091
    restart: unless-stopped
    environment:
      - TZ=Europe/Madrid
      - PUID=1000
      - GUID=1000
    healthcheck:
      disable: true
networks:
  proxy:
    external: true
```

## configuration.yml

```yml
---
###############################################################
#                   Authelia configuration                    #
###############################################################

server:
  host: 0.0.0.0
  port: 9091
log:
  level: info
  format: text
  file_path: /var/log/authelia/authelia.log
theme: dark
# This secret can also be set using the env variables AUTHELIA_JWT_SECRET_FILE
jwt_secret: long_string
default_redirection_url: https://auth.domain.com
totp:
  issuer: authelia.com

authentication_backend:
  disable_reset_password: true
  file:
    path: /config/users_database.yml
    password:
      algorithm: argon2id
      iterations: 1
      salt_length: 16
      parallelism: 8
      memory: 64

access_control:
  default_policy: deny
  rules:
    - domain:
      - "*.domain.com"
      policy: one_factor
      subject: "group:admins"

    - domain: send.domain.com
      policy: two_factor
      subject: "user:anon"

session:
  name: authelia_session
  # This secret can also be set using the env variables AUTHELIA_SESSION_SECRET_FILE
  secret: long_secret
  expiration: 8h
  inactivity: 2h
  domain: domain.com  # Should match whatever your root protected domain is

regulation:
  max_retries: 3
  find_time: 120
  ban_time: 300

storage:
  encryption_key: a_very_important_secret # Now required
  local:
    path: /config/db.sqlite3

notifier:
  filesystem:
      filename: /config/notification.txt
...
```

## users_database.yml

```yml
---
###############################################################
#                         Users Database                      #
###############################################################

# This file can be used if you do not have an LDAP set up.

# List of users
users:
  username:
    displayname: "Your Name"
    # Password is Authelia
    password: "$argon2id$v=19$m=65536,t=1,p=8$cUI4a0E3L1laYnRDUXl3Lw$ZsdsrdadaoVIaVj8NltA8x4qVOzT+/r5GF62/bT8OuAs" 
    email: you@example.com
    groups:
      - admins
      - dev
...
```
