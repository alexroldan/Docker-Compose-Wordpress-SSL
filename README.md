# Docker-Compose Wordpress SSL
 Official docker wordpress container with SSL in Windows 10/11

[https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)

**docker-compose.yml**
`
version: '3.9'
services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - 443:443
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: password
    volumes:
      - ./wordpress:/var/www/html
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: password
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
volumes:
  wordpress:
  db:
`

Create your docker-compose file and run `docker-compose up -d`

----------
# COPY FILES FROM WORDPRESS CONTAINER
Once the container is running, copy these 2 important files
- docker-entrypoint.sh `c:\ docker cp wordpress_wordpress_1:/usr/local/bin/docker-entrypoint.sh .`
- default-ssl.conf `c:\ docker cp wordpress_wordpress_1:/etc/apache2/sites-available/default-ssl.conf .`


After copying the files, shutdown the container 
`
docker-compose down
`
and update wordpress volumes in your docker-compose.yml file
`
    volumes:
      - ./wordpress:/var/www/html
      - ./certs:/etc/ssl/certs:ro
      - ./default-ssl.conf:/etc/apache2/sites-available/default-ssl.conf:ro
      - ./docker-entrypoint.sh:/usr/local/bin/docker-entrypoint.sh:ro
`

`./certs` - this is where your certificate will reside
`./default-ssl.conf` - this is where the 443 configuration
`./docker-entrypoint` - this run first before the web server starts

----------

# install mkcert in windows via choco install mkcert
follow the instruction on how to install mkcert on your host machine (windows)
[https://github.com/FiloSottile/mkcert](https://github.com/FiloSottile/mkcert)

Run the command to create self-signed certificate, follow through
`c:\ mkcert -install custom.domain.local` 

move the files created by mkcert in `./certs`
- custom.domain.local.pem
- custom.domain.local-key.pem

Check the certificate in windows (located in MMC Certificate / current user)
- run mmc
- File -> Add/Remove Snap in
- Choose certificate -> add -> press Ok
- Select `My user account` -> finish

Under the Trusted Root Certification Authorities -> Certificates, locate the certificates created by mkcert
you should find it there

----------
# Resolve custom domain name in browser via windows hosts
Modify the file `C:\Windows\System32\drivers\etc\hosts`
add this at the bottom `127.0.0.1 custom.domain.local`

----------
# Modify the 2 files
change the last line of `docker-entrypoint.sh`
`
a2enmod ssl
a2ensite default-ssl
service apache2 restart
service apache2 stop
exec "$@"
`

modify the `default-ssl.conf`
`
ServerName custom.domain.local
SSLCertificateFile	/etc/ssl/certs/custom.domain.local.pem
SSLCertificateKeyFile /etc/ssl/certs/custom.domain.local-key.pem
`

----------
# YOU ARE GOOD TO GO
run the container
`
docker-compose up -d
`

visit your wordpress site at `https://custom.domain.local`

You now have SSL enable wordpress site. Enjoy!














