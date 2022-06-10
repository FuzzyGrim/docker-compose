# Firefly III
## Docker-compose (Change MYSQL_DATABASE)
```
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
      - MYSQL_USER=firefly
      - MYSQL_PASSWORD=${SQL_PASS}
      - MYSQL_DATABASE=firefly
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
## .env (Change SITE_OWNER, APP_KEY, DEFAULT_LOCALE, TZ, DB_PASSWORD)
```
# You can leave this on "local". If you change it to production most console commands will ask for extra confirmation.
# Never set it to "testing".
APP_ENV=local

# Set to true if you want to see debug information in error screens.
APP_DEBUG=false

# This should be your email address.
# If you use Docker or similar, you can set this variable from a file by using SITE_OWNER_FILE
SITE_OWNER=mail@domain.com

# The encryption key for your sessions. Keep this very secure.
# Change it to a string of exactly 32 chars or use something like `php artisan key:generate` to generate it.
# If you use Docker or similar, you can set this variable from a file by using APP_KEY_FILE
APP_KEY=SomeRandomStringOf32CharsExactly

# Firefly III will launch using this language (for new users and unauthenticated visitors)
# For a list of available languages: https://github.com/firefly-iii/firefly-iii/tree/main/resources/lang
#
# If text is still in English, remember that not everything may have been translated.
DEFAULT_LANGUAGE=en_US

# The locale defines how numbers are formatted.
# by default this value is the same as whatever the language is.
DEFAULT_LOCALE=es_ES

# Change this value to your preferred time zone.
# Example: Europe/Amsterdam
# For a list of supported time zones, see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
TZ=Europe/Madrid

# TRUSTED_PROXIES is a useful variable when using Docker and/or a reverse proxy.
# Set it to ** and reverse proxies work just fine.
TRUSTED_PROXIES=

# The log channel defines where your log entries go to.
# Several other options exist. You can use 'single' for one big fat error log (not recommended).
# Also available are 'syslog', 'errorlog' and 'stdout' which will log to the system itself.
# A rotating log option is 'daily', creates 5 files that (surprise) rotate.
# A cool option is 'papertrail' for cloud logging
# Default setting 'stack' will log to 'daily' and to 'stdout' at the same time.
LOG_CHANNEL=stack

#
# Used when logging to papertrail:
#
PAPERTRAIL_HOST=
PAPERTRAIL_PORT=

# Log level. You can set this from least severe to most severe:
# debug, info, notice, warning, error, critical, alert, emergency
# If you set it to debug your logs will grow large, and fast. If you set it to emergency probably
# nothing will get logged, ever.
APP_LOG_LEVEL=notice

# Audit log level.
# Set this to "emergency" if you dont want to store audit logs, leave on info otherwise.
AUDIT_LOG_LEVEL=info

# Database credentials. Make sure the database exists. I recommend a dedicated user for Firefly III
# For other database types, please see the FAQ: https://docs.firefly-iii.org/support/faq
# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
# Use "pgsql" for PostgreSQL
# Use "mysql" for MySQL and MariaDB.
# Use "sqlite" for SQLite.
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=firefly
DB_USERNAME=firefly
DB_PASSWORD=secret_firefly_password
# leave empty or omit when not using a socket connection
DB_SOCKET=

# MySQL supports SSL. You can configure it here.
# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
MYSQL_USE_SSL=false
MYSQL_SSL_VERIFY_SERVER_CERT=true
# You need to set at least of these options
MYSQL_SSL_CAPATH=/etc/ssl/certs/
MYSQL_SSL_CA=
MYSQL_SSL_CERT=
MYSQL_SSL_KEY=
MYSQL_SSL_CIPHER=

# PostgreSQL supports SSL. You can configure it here.
# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
PGSQL_SSL_MODE=prefer
PGSQL_SSL_ROOT_CERT=null
PGSQL_SSL_CERT=null
PGSQL_SSL_KEY=null
PGSQL_SSL_CRL_FILE=null

# more PostgreSQL settings
PGSQL_SCHEMA=public

# If you're looking for performance improvements, you could install memcached or redis
CACHE_DRIVER=file
SESSION_DRIVER=file

# If you set either of the options above to 'redis', you might want to update these settings too
# If you use Docker or similar, you can set REDIS_HOST_FILE, REDIS_PASSWORD_FILE or
# REDIS_PORT_FILE to set the value from a file instead of from an environment variable

# can be tcp, unix or http
REDIS_SCHEME=tcp

# use only when using 'unix' for REDIS_SCHEME. Leave empty otherwise.
REDIS_PATH=

# use only when using 'tcp' or 'http' for REDIS_SCHEME. Leave empty otherwise.
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=null

# always use quotes and make sure redis db "0" and "1" exists. Otherwise change accordingly.
REDIS_DB="0"
REDIS_CACHE_DB="1"

# Cookie settings. Should not be necessary to change these.
# If you use Docker or similar, you can set COOKIE_DOMAIN_FILE to set
# the value from a file instead of from an environment variable
# Setting samesite to "strict" may give you trouble logging in.
COOKIE_PATH="/"
COOKIE_DOMAIN=
COOKIE_SECURE=false
COOKIE_SAMESITE=lax

# If you want Firefly III to email you, update these settings
# For instructions, see: https://docs.firefly-iii.org/advanced-installation/email
# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
MAIL_MAILER=log
MAIL_HOST=null
MAIL_PORT=2525
MAIL_FROM=changeme@example.com
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

# Other mail drivers:
# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
MAILGUN_DOMAIN=
MAILGUN_SECRET=


# If you are on EU region in mailgun, use api.eu.mailgun.net, otherwise use api.mailgun.net
# If you use Docker or similar, you can set this variable from a file by appending it with _FILE
MAILGUN_ENDPOINT=api.mailgun.net

# If you use Docker or similar, you can set these variables from a file by appending them with _FILE
MANDRILL_SECRET=
SPARKPOST_SECRET=

# Firefly III can send you the following messages.
SEND_REGISTRATION_MAIL=true
SEND_ERROR_MESSAGE=true
SEND_LOGIN_NEW_IP_WARNING=true

# These messages contain (sensitive) transaction information:
SEND_REPORT_JOURNALS=true

# Set this value to true if you want to set the location
# of certain things, like transactions. Since this involves an external service, it's optional
# and disabled by default.
ENABLE_EXTERNAL_MAP=false

# The map will default to this location:
MAP_DEFAULT_LAT=51.983333
MAP_DEFAULT_LONG=5.916667
MAP_DEFAULT_ZOOM=6

#
# Firefly III authentication settings
#

#
# Firefly III supports a few authentication methods:
# - 'web' (default, uses built in DB)
# - 'ldap'
# - 'remote_user_guard' for Authelia etc
 Read more about these settings in the documentation.
# https://docs.firefly-iii.org/advanced-installation/authentication
AUTHENTICATION_GUARD=web

#
# Your LDAP server may speak a dialect. You can choose between 'OpenLDAP' and 'ActiveDirectory'
# Anything else defaults to 'ActiveDirectory'
#
LDAP_DIALECT=OpenLDAP

#
# LDAP connection settings:
#
LDAP_HOST=ldap.yourserver.com
LDAP_PORT=389
LDAP_TIMEOUT=5
LDAP_SSL=false
LDAP_TLS=false

LDAP_BASE_DN="o=something,dc=site,dc=com"
LDAP_USERNAME="uid=X,ou=,o=,dc=something,dc=com"
LDAP_PASSWORD=super_secret

LDAP_AUTH_FIELD=uid

#
# If you wish to only authenticate users from a specific group, use the base DN above.
#
# If you require extra/special filters please use the LDAP_EXTRA_FILTER with a valid DN.
#
# The extra filter will only be applied after the user is authenticated.
#
LDAP_EXTRA_FILTER=

#
# Remote user guard settings
#
AUTHENTICATION_GUARD_HEADER=REMOTE_USER
AUTHENTICATION_GUARD_EMAIL=

#
# Extra authentication settings
#
CUSTOM_LOGOUT_URL=

# You can disable the X-Frame-Options header if it interferes with tools like
# Organizr. This is at your own risk. Applications running in frames run the risk
# of leaking information to their parent frame.
DISABLE_FRAME_HEADER=false

# You can disable the Content Security Policy header when you're using an ancient browser
# or any version of Microsoft Edge / Internet Explorer (which amounts to the same thing really)
# This leaves you with the risk of not being able to stop XSS bugs should they ever surface.
# This is at your own risk.
DISABLE_CSP_HEADER=false

# If you wish to track your own behavior over Firefly III, set valid analytics tracker information here.
# Nobody uses this except for me on the demo site. But hey, feel free to use this if you want to.
# Do not prepend the TRACKER_URL with http:// or https://
# The only tracker supported is Matomo.
# You can set the following variables from a file by appending them with _FILE:
TRACKER_SITE_ID=
TRACKER_URL=

#
# Firefly III supports webhooks. These are security sensitive and must be enabled manually first.
#
ALLOW_WEBHOOKS=false

#
# The static cron job token can be useful when you use Docker and wish to manage cron jobs.
# 1. Set this token to any 32-character value (this is important!).
# 2. Use this token in the cron URL instead of a user's command line token.
#
# For more info: https://docs.firefly-iii.org/firefly-iii/advanced-installation/cron/
#
# You can set this variable from a file by appending it with _FILE
#
STATIC_CRON_TOKEN=

# You can fine tune the start-up of a Docker container by editing these environment variables.
# Use this at your own risk. Disabling certain checks and features may result in lost of inconsistent data.
# However if you know what you're doing you can significantly speed up container start times.
# Set each value to true to enable, or false to disable.

# Check if the SQLite database exists. Can be skipped if you're not using SQLite.
# Won't significantly speed up things.
DKR_CHECK_SQLITE=true

# Run database creation and migration commands. Disable this only if you're 100% sure the DB exists
# and is up to date.
DKR_RUN_MIGRATION=true

# Run database upgrade commands. Disable this only when you're 100% sure your DB is up-to-date
# with the latest fixes (outside of migrations!)
DKR_RUN_UPGRADE=true

# Verify database integrity. Includes all data checks and verifications.
# Disabling this makes Firefly III assume your DB is intact.
DKR_RUN_VERIFY=true

# Run database reporting commands. When disabled, Firefly III won't go over your data to report current state.
# Disabling this should have no impact on data integrity or safety but it won't warn you of possible issues.
DKR_RUN_REPORT=true

# Generate OAuth2 keys.
# When disabled, Firefly III won't attempt to generate OAuth2 Passport keys. This won't be an issue, IFF (if and only if)
# you had previously generated keys already and they're stored in your database for restoration.
DKR_RUN_PASSPORT_INSTALL=true

# Leave the following configuration vars as is.
# Unless you like to tinker and know what you're doing.
APP_NAME=FireflyIII
ADLDAP_CONNECTION=default
BROADCAST_DRIVER=log
QUEUE_DRIVER=sync
CACHE_PREFIX=firefly
PUSHER_KEY=
IPINFO_TOKEN=
PUSHER_SECRET=
PUSHER_ID=
DEMO_USERNAME=
DEMO_PASSWORD=
IS_HEROKU=false
FIREFLY_III_LAYOUT=v1

#
# If you have trouble configuring your Firefly III installation, DON'T BOTHER setting this variable.
# It won't work. It doesn't do ANYTHING. Don't believe the lies you read online. I'm not joking.
# This configuration value WILL NOT HELP.
#
# Notable exception to this rule is Synology, which, according to some users, will use APP_URL to rewrite stuff.
#
# This variable is ONLY used in some of the emails Firefly III sends around. Nowhere else.
# So when configuring anything WEB related this variable doesn't do anything. Nothing
#
# If you're stuck I understand you get desperate but look SOMEWHERE ELSE.
#
APP_URL=http://localhost
```

## .fidi.env (Change FIREFLY_III_URL, TZ, FIREFLY_III_ACCESS_TOKEN)

```
#
# Where is Firefly III?
#
# 1) Make sure you ADD http:// or https://
# 2) Make sure you REMOVE any trailing slash from the end of the URL.
# 3) In case of Docker, refer to the internal IP of your Firefly III installation.
#
# This value is not mandatory. But it is very useful.
#
# This variable can be set from a file if you append it with _FILE
#
FIREFLY_III_URL=http://192.168.1.122:8003

#
# Imagine Firefly III can be reached at "http://172.16.0.2:8082" (internal Docker network or something).
# But you have a fancy URL: "https://personal-finances.bill.microsoft.com/"
#
# In those cases, you can overrule the URL so when the data importer links back to Firefly III, it uses the correct URL.
#
# 1) Make sure you ADD http:// or https://
# 2) Make sure you REMOVE any trailing slash from the end of the URL.
#
# IF YOU SET THIS VALUE, YOU MUST ALSO SET THE FIREFLY_III_URL
#
# This variable can be set from a file if you append it with _FILE
#
VANITY_URL=

#
# Set your Firefly III Personal Access Token
# You can find create a Personal Access Token on the /profile page (at the bottom)
#
# - Do not use the "command line token". That's the WRONG one.
# - Do not use "APP_KEY". That's the WRONG one.
#
# This value is not mandatory to set. Instructions will follow if you omit this field.
#
# This variable can be set from a file if you append it with _FILE
#
FIREFLY_III_ACCESS_TOKEN=

#
# You can also use a public client ID. This is available in Firefly III 5.4.0-alpha.3 and higher.
# This is a number (1, 2, 3). If you use the client ID, you can leave the access token empty and vice versa.
#
# This value is not mandatory to set. Instructions will follow if you omit this field.
#
# This variable can be set from a file if you append it with _FILE
#
FIREFLY_III_CLIENT_ID=

#
# Nordigen information.
# The key and ID can be set from a file if you append it with _FILE
#
NORDIGEN_ID=
NORDIGEN_KEY=
NORDIGEN_SANDBOX=false

#
# Spectre information
#
# The ID and secret can be set from a file if you append it with _FILE
SPECTRE_APP_ID=
SPECTRE_SECRET=

#
# Use cache. No need to do this.
#
USE_CACHE=false

#
# If set to true, the data import will not complain about running into duplicates.
# This will give you cleaner import mails if you run regular imports.
#
# Of course, if something goes wrong *because* the transaction is a duplicate you will
# NEVER know unless you start digging in your log files. So be careful with this.
#
IGNORE_DUPLICATE_ERRORS=false

#
# Auto import settings. Due to security constraints, you MUST enable each feature individually.
# You must also set a secret. The secret is used for the web routes.
#
# The auto-import secret must be a string of at least 16 characters.
# Visit this page for inspiration: https://www.random.org/passwords/?num=1&len=16&format=html&rnd=new
#
# Submit it using ?secret=X
#
# This variable can be set from a file if you append it with _FILE
#
AUTO_IMPORT_SECRET=

#
# Is the /autoimport even endpoint enabled?
# By default it's disabled, and the secret alone will not enable it.
#
CAN_POST_AUTOIMPORT=false

#
# Is the /autoupload endpoint enabled?
# By default it's disabled, and the secret alone will not enable it.
#
CAN_POST_FILES=false

#
# Import directory white list. You need to set this before the auto importer will accept a directory to import from.
#
# This variable can be set from a file if you append it with _FILE
#
IMPORT_DIR_WHITELIST=

#
# When you're running Firefly III under a (self-signed) certificate,
# the data importer may have trouble verifying the TLS connection.
#
# You have a few options to make sure the data importer can connect
# to Firefly III:
# - 'true': will verify all certificates. The most secure option and the default.
# - 'file.pem': refer to a file (you must provide it) to your custom root or intermediate certificates.
# - 'false': will verify NO certificates. Not very secure.
VERIFY_TLS_SECURITY=true

#
# If you want, you can set a directory here where the data importer will look for import configurations.
# This is a separate setting from the /import directory that the auto-import uses.
# Setting this variable isn't necessary. The default value is "storage/configurations".
#
# This variable can be set from a file if you append it with _FILE
#
JSON_CONFIGURATION_DIR=

#
# Time out when connecting with Firefly III.
# ~@*10 seconds is usually fine.
#
CONNECTION_TIMEOUT=31.41

# The following variables can be useful when debugging the application
APP_ENV=local
APP_DEBUG=false
LOG_CHANNEL=stack

# Log level. You can set this from least severe to most severe:
# debug, info, notice, warning, error, critical, alert, emergency
# If you set it to debug your logs will grow large, and fast. If you set it to emergency probably
# nothing will get logged, ever.
LOG_LEVEL=debug

# TRUSTED_PROXIES is a useful variable when using Docker and/or a reverse proxy.
# Set it to ** and reverse proxies work just fine.
TRUSTED_PROXIES=

#
# Time zone
#
TZ=Europe/Madrid

#
# Use ASSET_URL when you're running the data importer in a sub-directory.
#
ASSET_URL=

#
# Email settings.
# The data importer can send you a message with all errors, warnings and messages
# after a successful import. This is disabled by default
#
ENABLE_MAIL_REPORT=false

#
# Force Firefly III URL to be secure?
#
#
EXPECT_SECURE_URL=false

# If enabled, define which mailer you want to use.
# Options include: smtp, mailgun, postmark, sendmail, log, array
# Amazon SES is not supported.
# log = drop mails in the logs instead of sending them
# array = debug mailer that does nothing.
MAIL_MAILER=


# where to send the report?
MAIL_DESTINATION=noreply@example.com

# other mail settings
# These variables can be set from a file if you append it with _FILE
MAIL_FROM_ADDRESS=noreply@example.com
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=username
MAIL_PASSWORD=password
MAIL_ENCRYPTION=null

# Extra settings depending on your mail configuration above.
# These variables can be set from a file if you append it with _FILE
MAILGUN_DOMAIN=
MAILGUN_SECRET=
MAILGUN_ENDPOINT=
POSTMARK_TOKEN=

#
# You probably won't need to change these settings.
#
BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120
IS_EXTERNAL=false

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# always use quotes
REDIS_DB="0"
REDIS_CACHE_DB="1"

# The only tracker supported is Matomo.
# This is used on the public instance over at https://data-importer.firefly-iii.org
TRACKER_SITE_ID=
TRACKER_URL=


APP_NAME=DataImporter

#
# The APP_URL environment variable is NOT used anywhere.
# Don't bother setting it to fix your reverse proxy problems. It won't help.
# Don't open issues telling me it doesn't help because it's not supposed to.
# Laravel uses this to generate links on the command line, which is a feature the CVS importer does not use.
#
APP_URL=http://localhost
```

## Backup
Cronjob:
```
0 14 * * 6 docker stop firefly && docker exec firefly_db mysqldump --single-transaction -h localhost -u firefly -p${SQL_PASS} firefly > /data/firefly/db_bak/firefly-sqlbkp_$(date '+\%Y\%m\%d-\%H\%M').sql && docker start firefly
```