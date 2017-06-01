## wonderfall/nextcloud


[![](https://images.microbadger.com/badges/version/wonderfall/nextcloud.svg)](http://microbadger.com/images/wonderfall/nextcloud "Get your own version badge on microbadger.com") [![](https://images.microbadger.com/badges/image/wonderfall/nextcloud.svg)](http://microbadger.com/images/wonderfall/nextcloud "Get your own image badge on microbadger.com")

[![](https://images.microbadger.com/badges/version/wonderfall/nextcloud:daily.svg)](https://microbadger.com/images/wonderfall/nextcloud:daily "Get your own version badge on microbadger.com") [![](https://images.microbadger.com/badges/image/wonderfall/nextcloud:daily.svg)](https://microbadger.com/images/wonderfall/nextcloud:daily "Get your own image badge on microbadger.com")

[![](https://images.microbadger.com/badges/version/wonderfall/nextcloud:11.0.svg)](https://microbadger.com/images/wonderfall/nextcloud:11.0 "Get your own version badge on microbadger.com") [![](https://images.microbadger.com/badges/image/wonderfall/nextcloud:11.0.svg)](https://microbadger.com/images/wonderfall/nextcloud:11.0 "Get your own image badge on microbadger.com")

![](https://s32.postimg.org/69nev7aol/Nextcloud_logo.png)

**This dockerfile is modified from wonderfall/nextcloud. For more information click [here](  https://github.com/Wonderfall/dockerfiles/blob/master/nextcloud/README.md#wonderfallnextcloud)**


### Docker-compose
I advise you to use [docker-compose](https://docs.docker.com/compose/), which is a great tool for managing containers. You can create a `docker-compose.yml` with the following content (which must be adapted to your needs) and then run `docker-compose up -d nextcloud-db`, wait some 15 seconds for the database to come up, then run everything with `docker-compose up -d`, that's it! On subsequent runs,  a single `docker-compose up -d` is sufficient!

### Port
- **8888** : HTTP Nextcloud port.


### IP
- **IP** : Nextcloud server ip address. to find ip address of your nextcloud server `docker inspect DOCKERIMAGE | grep IPAddress | grep -Eo '[0-9.]+' | head -n1` 


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

### Reverse proxy
I use nginx on the host machine. Headers are already sent by the container, including HSTS, so there's no need to add them again. **It is strongly recommended to use Nextcloud through an encrypted connection (HTTPS).** 
so if you're using nginx, below is the configuration.

```
server {
  listen 8000;
  server_name example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 4430 ssl http2;
  server_name example.com;

  ssl_certificate /certs/example.com.crt;
  ssl_certificate_key /certs/example.com.key;

  include /etc/nginx/conf/ssl_params.conf;

  client_max_body_size 10G; # change this value it according to $UPLOAD_MAX_SIZE

  location / {
    proxy_pass http://nextcloud:8888;
    include /etc/nginx/conf/proxy_params;
  }
}
```

