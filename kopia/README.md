# Kopia

```yml
version: '3.7'
services:
    kopia:
        image: kopia/kopia:latest
        hostname: metal
        container_name: kopia
        restart: unless-stopped
        ports:
            - 51515:51515
        environment:
            - KOPIA_PASSWORD=${KOPIA_REPOSITORY_PASSWORD}
            - KOPIA_PERSIST_CREDENTIALS_ON_CONNECT=true
            - TZ=Europe/Madrid
        volumes:
            - ./config:/app/config
            - ./cache:/app/cache
            - ./logs:/app/logs
            - /data:/app/data:ro
            - /backup:/app/backup
        command:
            - server
            - start
            - --insecure
            - --address=0.0.0.0:51515
            - --server-username=kopia@metal
            - --server-password=${KOPIA_USER_PASSWORD}
```

## Create a repository

When you first start Kopia, you need to create a repository and it will ask for a password. You will need to use the $KOPIA_REPOSITORY_PASSWORD environment variable for this.
