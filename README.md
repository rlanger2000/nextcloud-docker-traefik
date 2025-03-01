# Nextcloud docker-compose for Traefik
Intended for use with an existing Traefik reverse proxy.

**Absolutely make sure you set the MySQL passwords, your domain, traefik entrypoint and cert-resolver !**\
After the initial setup, you can change the ``Background jobs`` setting in the admin panel to ``Cron``.\
\
![image](https://github.com/user-attachments/assets/dba2418a-fb24-4637-8431-93e2695ed25b)

## docker-compose

```yaml
# Set MySQL passwords, domain, entrypoint and cert-resolver

services:
  db:
    image: mariadb:10.6
    restart: unless-stopped
    networks:
      - nextcloud
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD='PASSWORD SUPER SECURE (recovery only)' # Root password, only for recovery really
      - MYSQL_PASSWORD='PASSWORD' # Password for the nextcloud database
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud:apache
    restart: unless-stopped
    networks:
      - nextcloud
    volumes:
      - nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD='PASSWORD' # Password for the nextcloud database, same as above
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
      # ↓ This was a pain to figure out. If you don't set this, clicking any button or logging in/out will redirect to HTTP (no S) and return 404
      - OVERWRITEPROTOCOL=https 
    depends_on:
      - db
    labels:
      - "traefik.enable=true"
      
      # Webinterface Router                       # Domain goes here #
      - "traefik.http.routers.nextcloud.rule=Host(`YOUR.COOL.DOMAIN`)"
      - "traefik.http.routers.nextcloud.entrypoints=https"
      - "traefik.http.routers.nextcloud.tls=true"
      - "traefik.http.routers.nextcloud.tls.certresolver=lets-encrypt"
      - "traefik.http.routers.nextcloud.service=nextcloud-http"

      # Service
      - "traefik.http.services.nextcloud-http.loadbalancer.server.port=80"

  cron:
    image: nextcloud:apache
    restart: unless-stopped
    networks:
      - nextcloud
    volumes:
      - nextcloud:/var/www/html:z
    entrypoint: /cron.sh
    depends_on:
      - db

  networks:
    nextcloud:
      name: nextcloud
  
  volumes:
    nextcloud:
    db:
```
