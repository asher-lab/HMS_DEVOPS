#  Implement DevOps process and principles on Hospital Management System

##  System Requirements: 
1. These are the necessary components in order to implement DevOps processes in the system:
	- Apache HTTP Server
	- PHPMyAdmin
	- Docker Runtime
	- Jenkins

## 🧊 Development
Step 1: Pull the Repo and Create a New Branch (e.g. bugfix or feature branch)
```
commands
```
Step 2: Commit Changes to Local Machine.
```
commands
```

## 🧊Push Code Changes to Repository
Step 1: Push your code changes to `YOUR-BRANCH`
```
commands
```
Step 2: Merge `YOUR-BRANCH` to `MAIN`
## 🧊Webhooks
Build the project when triggered using Jenkins
```
commands
```
## 🧊Testing 
## 🧊Versioning of Images


## 🧊 Build Image (Containerization)
```
# Create a Directory for Project:
mkdir Proj && cd Proj

# Clone the repository (Also one can pull so only the change are downloaded):
git clone https://github.com/QuantumBit-Softwares/Hospital-Management-System.git

cd Hospital-Management-System

#  Install HTTPD Apache
systemctl start docker
docker pull httpd

# Prepare the location of the files
mkdir -p /usr/local/apache2/htdocs/
cp -r public-html /usr/local/apache2/htdocs/
firewall-cmd --zone=public --add-port=41602/tcp --permanent
systemctl reload firewalld
docker run -dit --name my-apache-app -p 8081:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4

#Navigate to:
http://34.234.35.40:8081/

#get httpd.conf to edit:
curl -k -F "file=@httpd.conf" https://file.io

# Set Apache to load index.php by default
https://stackoverflow.com/questions/2384423/index-php-not-loading-by-default
docker exec -it -u 0 a2a11daa3d07 sh

#copy file
docker cp httpd.conf <yourcontainer_name>:/path/to/httpd.conf

#apache restart
sudo docker exec -it YOUR_CONTAINER_NAME apachectl graceful

```
### A .  Create a  `Dockerfile`  in your project ( Apache HTTP Server )
```dockerfile
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
```
Then, run the commands to build and run the Docker image:

```console
$ docker build -t hospital-management-system:1.0 .
$ docker run -dit --name hospital-management-system -p 8081:80 hospital-management-system:1.0
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
```
docker run --name mysql_db_server -e MYSQL_ROOT_PASSWORD=newpass -d mysql:5.6
docker run --name phpmyadmin -d --link mysql_db_server:db -p 8080:80 phpmyadmin
```
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

```
version: '3'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}

services:
  mysql:
    image: mysql:5.6
    ports:
      - 3307:3306
    environment:
      - MYSQL_ROOT_PASSWORD=newpass
      - 'MYSQL_DATABASE=myhmsdb'
    volumes:
      - ./mysqldump:/docker-entrypoint-initdb.d # mysqldump folder should contain db.sql
    restart: unless-stopped

  my-app-crud:
    image: my-app-crud:1.0
    restart: unless-stopped
    network_mode: "host"

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
#    expose:
#      - 9100
    ports:
     - "9091:9100"

    networks:
      - monitoring

  promcontainer:
    image: "prom/prometheus:latest"
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/etc/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
#    expose:
#      - 9090
    networks:
     - monitoring
    ports:
     - "9090:9090"

  grafanacontainer:
    image: "grafana/grafana"
    ports:
     - "3000:3000"
    networks:
     - monitoring
```
## 🧊 Security Scan
https://docs.docker.com/engine/scan/

## 🧊 Publish

## 🧊 Deploy
- deploying the image on compute instances
