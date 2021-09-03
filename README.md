#  Implement DevOps process and principles on Hospital Management System

##  System Requirements: 
1. These are the necessary components in order to implement DevOps processes in the system:
	- Apache HTTP Server
	- PHPMyAdmin
	- Docker Runtime
	- Jenkins
## 🧊 Containerization
### A .  Create a  `Dockerfile`  in your project ( Apache HTTP Server )
```dockerfile
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
```
Then, run the commands to build and run the Docker image:

```console
$ docker build -t my-apache2 .
$ docker run -dit --name my-running-app -p 8080:80 my-apache2
```

Visit  [http://localhost:8080](http://localhost:8080/)  and you will see It works!
#### SSL/HTTPS

If you want to run your web traffic over SSL, the simplest setup is to  `COPY`  or mount (`-v`) your  `server.crt`  and  `server.key`  into  `/usr/local/apache2/conf/`  and then customize the  `/usr/local/apache2/conf/httpd.conf`  by removing the comment symbol from the following lines:

```apacheconf
...
#LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
...
#LoadModule ssl_module modules/mod_ssl.so
...
#Include conf/extra/httpd-ssl.conf
...
```

The  `conf/extra/httpd-ssl.conf`  configuration file will use the certificate files previously added and tell the daemon to also listen on port 443. Be sure to also add something like  `-p 443:443`  to your  `docker run`  to forward the https port.

This could be accomplished with a  `sed`  line similar to the following:

```dockerfile
RUN sed -i \
        -e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
        -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
        conf/httpd.conf
```

The previous steps should work well for development, but we recommend customizing your conf files for production, see  [httpd.apache.org](https://httpd.apache.org/docs/2.4/ssl/ssl_faq.html)  for more information about SSL setup.
### B.  Create a  `DockeCompose`  in your project ( Database Server )
### Usage with docker-compose and arbitrary server

This will run phpMyAdmin with arbitrary server - allowing you to specify MySQL/MariaDB server on login page.

... via  [`docker stack deploy`](https://docs.docker.com/engine/reference/commandline/stack_deploy/)  or  [`docker-compose`](https://github.com/docker/compose)

Example  `stack.yml`  for  `phpmyadmin`:

```yaml
version: '3.1'

services:
  db:
    image: mariadb:10.3
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: notSecureChangeMe

  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY=1
```
### C.  Create a  `DockeCompose`  in your project ( Database Server + User Interface)
🌈 Ensure Docker Images are present
🌈 Write the Docker Compose

