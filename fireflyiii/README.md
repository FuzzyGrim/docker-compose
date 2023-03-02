# Firefly III

## Docker-compose (Change MYSQL_DATABASE)

```yml
version: '3.3'

services:
  app:
    container_name: firefly
    image: fireflyiii/core:latest
    restart: always
    volumes:
      - /data/firefly/upload:/var/www/html/storage/upload
    env_file: .env
    ports:
      - 8003:8080
    depends_on:
      - db
  db:
    container_name: firefly_db
    image: mariadb
    hostname: fireflyiiidb
    restart: always
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=yes
      - MYSQL_USER=${DB_USERNAME}
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_DATABASE=${DB_DATABASE}
    volumes:
      - /data/firefly/database:/var/lib/mysql
  fidi:
    container_name: firefly_fidi
    restart: always
    image: fireflyiii/data-importer:latest
    env_file: .fidi.env
    ports:
      - 8081:8080
    depends_on:
      - app
```

## .env

Download the example env file from: [https://raw.githubusercontent.com/firefly-iii/firefly-iii/main/.env.example](https://raw.githubusercontent.com/firefly-iii/firefly-iii/main/.env.example) and rename it to `.env`.

Then edit the following variables:

- SITE_OWNER
- APP_KEY
- DEFAULT_LOCALE
- TZ
- DB_PASSWORD

## .fidi.env (Change FIREFLY_III_URL, TZ, FIREFLY_III_ACCESS_TOKEN)

Download the example env file from: [https://raw.githubusercontent.com/firefly-iii/data-importer/main/.env.example](https://raw.githubusercontent.com/firefly-iii/data-importer/main/.env.example) and rename it to `.fidi.env`.

Then edit the following variables:

- FIREFLY_III_URL
- FIREFLY_III_ACCESS_TOKEN

You can generate your own Personal Access Token on the Profile page. Login to your Firefly III instance, go to "Options" > "Profile" > "OAuth" and find "Personal Access Tokens". Create a new Personal Access Token by clicking on "Create New Token". Give it a recognizable name and press "Create". The Personal Access Token is pretty long.

## Backup

Cronjob:

```
0 14 * * * docker exec firefly_db mysqldump --single-transaction -h localhost -u firefly -pET6Ysh5TXt3t9QStjypeBYFNWuLJAw9U firefly > /data/firefly/db_bak/firefly-sqlbkp_$(date '+\%Y-\%m-\%d').sql
```
