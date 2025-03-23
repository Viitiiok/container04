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
2. Baza de date MariaDB 
3. Configurația necesară pentru a rula WordPress
4. Expozitia serverului pe portul 8000

# Executare Pas cu Pas

## Creați un depozit de cod sursă containers04 și clonați-l pe computerul dvs.
<img width="255" alt="Screenshot 2025-03-23 at 17 47 28" src="https://github.com/user-attachments/assets/0ee2e970-5bf4-4968-ba33-58ed6230fc2e" />
### Extragerea fișierelor de configurare apache2, php, mariadb din container
Dockerfile: 
FROM debian:latest

RUN apt-get update && \
   apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
   apt-get clean
<img width="1791" alt="Screenshot 2025-03-23 at 17 49 17" src="https://github.com/user-attachments/assets/0266cd6c-cabb-4e2b-a9b1-9653383bdf31" />
<img width="785" alt="Screenshot 2025-03-23 at 17 49 55" src="https://github.com/user-attachments/assets/44998523-8d8f-47d4-9228-1eaf419a7bdd" />
<img width="1051" alt="Screenshot 2025-03-23 at 17 50 43" src="https://github.com/user-attachments/assets/880abae7-b30e-44a1-ba1a-5195a90fe6d3" />
<img width="641" alt="Screenshot 2025-03-23 at 17 50 59" src="https://github.com/user-attachments/assets/964263bc-a466-4ca1-b61f-72d5960bbcb5" />
<img width="439" alt="Screenshot 2025-03-23 at 17 51 20" src="https://github.com/user-attachments/assets/b6f98683-0ba1-4552-a8f0-34f4d016878e" />

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
<img width="205" alt="Screenshot 2025-03-23 at 18 07 04" src="https://github.com/user-attachments/assets/73477d6f-b305-43c5-8361-5f9353ce2ff3" />

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
<img width="621" alt="Screenshot 2025-03-23 at 18 09 31" src="https://github.com/user-attachments/assets/2928f485-57c4-4a1a-be72-5fe22203b6ae" />

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
<img width="1789" alt="Screenshot 2025-03-23 at 18 24 00" src="https://github.com/user-attachments/assets/5907bfa0-ffaf-489b-9b17-edde3ef303a9" />
<img width="697" alt="Screenshot 2025-03-23 at 18 35 58" src="https://github.com/user-attachments/assets/3aba6103-5372-487b-bbd9-7cc232fb7837" />

## Crearea bazelor de date și a utilizatorului pentru WordPress
Crearea bazelor de date și a utilizatorului pentru WordPress se face în containerul apache2-php-mariadb. Pentru a face acest lucru, conectați-vă la containerul apache2-php-mariadb și executați comenzile:
<img width="821" alt="Screenshot 2025-03-23 at 18 37 28" src="https://github.com/user-attachments/assets/004491fc-0572-4550-85af-842e20b49d49" />

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
