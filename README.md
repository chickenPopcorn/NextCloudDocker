### Docker-compose
I advise you to use [docker-compose](https://docs.docker.com/compose/), which is a great tool for managing containers. You can create a `docker-compose.yml` with the following content (which must be adapted to your needs) and then run `docker-compose up -d nextcloud-db`, wait some 15 seconds for the database to come up, then run everything with `docker-compose up -d`, that's it! On subsequent runs,  a single `docker-compose up -d` is sufficient!

#### Docker-compose file
Don't copy/paste without thinking! It is a model so you can see how to do it correctly.

```
nextcloud:
  image: wonderfall/nextcloud
  links:
    - nextcloud-db:nextcloud-db   # If using MySQL
    - solr:solr                   # If using Nextant
    - redis:redis                 # If using Redis
  environment:
    - UID=1000
    - GID=1000
    - UPLOAD_MAX_SIZE=10G
    - APC_SHM_SIZE=128M
    - OPCACHE_MEM_SIZE=128
    - CRON_PERIOD=15m
    - TZ=Europe/Berlin
    - ADMIN_USER=admin            # Don't set to configure through browser
    - ADMIN_PASSWORD=admin        # Don't set to configure through browser
    - DOMAIN=localhost
    - DB_TYPE=mysql
    - DB_NAME=nextcloud
    - DB_USER=nextcloud
    - DB_PASSWORD=supersecretpassword
    - DB_HOST=nextcloud-db
  volumes:
    - /mnt/nextcloud/data:/data
    - /mnt/nextcloud/config:/config
    - /mnt/nextcloud/apps:/apps2
    - /mnt/nextcloud/themes:/nextcloud/themes

# If using MySQL
nextcloud-db:
  image: mariadb:10
  volumes:
    - /mnt/nextcloud/db:/var/lib/mysql
  environment:
    - MYSQL_ROOT_PASSWORD=supersecretpassword
    - MYSQL_DATABASE=nextcloud
    - MYSQL_USER=nextcloud
    - MYSQL_PASSWORD=supersecretpassword
    
# If using Nextant
solr:
  image: solr:6-alpine
  container_name: solr
  volumes:
    - /mnt/docker/solr:/opt/solr/server/solr/mycores
  entrypoint:
    - docker-entrypoint.sh
    - solr-precreate
    - nextant

# If using Redis
redis:
  image: redis:alpine
  container_name: redis
  volumes:
    - /mnt/docker/redis:/data
```

You can update everything with `docker-compose pull` followed by `docker-compose up -d`.
