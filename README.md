# container04

# Lucrarea de Laborator Nr. 4: Configurarea unui Mediu Web cu Apache, PHP și MariaDB în Docker

## Scopul Lucrării
După finalizarea acestei lucrări, studentul va fi capabil să configureze un container Docker care să găzduiască un site web bazat pe:
- Apache HTTP Server
- PHP
- MariaDB 

## Sarcina 
Crearea unui fișier Dockerfile pentru construirea unei imagini ce include:
1. Serverul Apache cu suport PHP
2. Baza de date MariaDB (cu volum dedicat)
3. Configurația necesară pentru a rula WordPress
4. Expozitia serverului pe portul **8000**

# Executare Pas cu Pas

## Creați un depozit de cod sursă containers04 și clonați-l pe computerul dvs.
### Extragerea fișierelor de configurare apache2, php, mariadb din container
Dockerfile: 
FROM debian:latest

RUN apt-get update && \
   apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
   apt-get clean

## Configurarea
### Fișierul de configurare apache2
1. 000-default.conf:
ServerName localhost
ServerAdmin victor76167@gmail.com
DirectoryIndex index.php index.html

2. apache2.conf:
ServerName localhost

3. php.ini:
error_log = /var/log/php_errors.log
memory_limit = 128M
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 120

4. 50-server.cnf:
log_error = /var/log/mysql/error.log

## Configurare Supervisor
[supervisord]
nodaemon=true
logfile=/dev/null
user=root

### apache2
[program:apache2]
command=/usr/sbin/apache2ctl -D FOREGROUND
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=root

### mariadb
[program:mariadb]
command=/usr/sbin/mariadbd --user=mysql
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=mysql

## Dockerfile Final
### Dockerfile:
ADD https://wordpress.org/latest.tar.gz /var/www/html/
RUN tar -xzf /var/www/html/latest.tar.gz -C /var/www/html/ --strip-components=1

COPY files/apache2/* /etc/apache2/
COPY files/php/php.ini /etc/php/8.2/apache2/
COPY files/mariadb/50-server.cnf /etc/mysql/mariadb.conf.d/
COPY files/supervisor/supervisord.conf /etc/supervisor/conf.d/

RUN mkdir /var/run/mysqld && chown mysql:mysql /var/run/mysqld

EXPOSE 80
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

## Construire și Rulare
docker build -t apache2-php-mariadb .
docker run -d -p 8000:80 -v mysql_data:/var/lib/mysql --name my-container apache2-php-mariadb

## Configurare WordPress
Accesam http://localhost:8000 și completam:
Database Name: wordpress
Username: wordpress
Password: wordpress
Host: localhost

## Răspunsuri la Întrebări:
Fișiere de configurare modificate:
000-default.conf, apache2.conf, php.ini, 50-server.cnf

DirectoryIndex în Apache:
Definește ordinea fișierelor index afișate automat.

Rolul wp-config.php:
Conține setările de conectare la baza de date și alte configurări critice WordPress.

post_max_size în PHP:
Limita maximă pentru datele trimise prin POST.

Dezavantaje ale imaginii:
Servicii multiple într-un singur container
Folosirea utilizatorului root
Lipsa optimizărilor de securitate

## Concluzii
Am configurat cu succes un mediu web complet în Docker, integrand Apache, PHP, MariaDB și WordPress. Am învățat să gestionez configurațiile într-un container, precum și importanța volumelor pentru date persistente.
