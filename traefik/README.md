# Traefik

Check my [guide](https://www.fuzzygrim.com/posts/exposing-services) on how to setup Traefik with Authelia.

## Docker compose
```yml
version: '3'

services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - 80:80
      - 443:443
    environment:
      - CF_API_EMAIL=${MAIL}
      - CF_API_KEY=${KEY}
      # be sure to use the correct one depending on if you are using a token or key
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/acme.json:/acme.json
      - ./data/config.yml:/config.yml:ro
      - /var/log/traefik:/var/log/traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=https"
      - "traefik.http.routers.traefik.rule=Host(`traefik.domain.com`)"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=user:password"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
      - "traefik.http.routers.traefik.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik.tls.domains[0].main=domain.com"
      - "traefik.http.routers.traefik.tls.domains[0].sans=*.domain.com"
      - "traefik.http.routers.traefik.service=api@internal"

networks:
  proxy:
    external: true
```

Then: `touch ./data/acme.json && chmod 600 ./data/acme.json`

## ./data/config.yml
```yml
http:
  middlewares:
    https-redirect:
      redirectScheme:
        scheme: https
        permanent: true
    authelia:
      forwardAuth:
        address: "http://authelia:9091/api/verify?rd=https://auth.domain.com"
    default-headers:
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 15552000
        customFrameOptionsValue: SAMEORIGIN
        customRequestHeaders:
          X-Forwarded-Proto: https
          X-Robots-Tag: "noindex,nofollow,nosnippet,noarchive,notranslate,noimageindex"

    secured:
      chain:
        middlewares:
        - default-headers
```

## ./data/traefik.yml
```yml
api:
  dashboard: true
  debug: true

entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
          permanent: true

  https:
    address: ":443"
    forwardedHeaders:
      trustedIPs:
        - "192.168.0.0/16"

serversTransport:
  insecureSkipVerify: true

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: /config.yml
certificatesResolvers:
  cloudflare:
    acme:
      email: ${MAIL}
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
log:
  level: "INFO"
  filePath: "/var/log/traefik/traefik.log"
accessLog:
  filePath: "/var/log/traefik/access.log"
```
