empezaremos con la configuracion de la ip siguiendo la estructura que ideamos (enlace doc con organizacion)

lo primero es hacer una actualizacion 

apt update
apt upgrade

ya actualizado connfiguramos la ip en este archivo /etc/network/interfaces

auto enp0s3
iface enp0s3 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.1 8.8.8.8

canbiamos el hostname podemos haerlo desde el archivo /etc/hostname o con este comando

hostnamectl set-hostname zabbix-server

y añadimos esto al /etc/hosts

192.168.1.10    zabbix-server

para que todos los cambios se hagan reiniciamos la maquina

ya podemos empezar añadiendo los repositorios antes debemos saber la version de nuestro sistema

aqui veremos la version:

cat /etc/apt/sources.list

con este comando descargamos el archivo con los repositorios

wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb

![alt text](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\wget.png)

y con este los instalamos

dpkg -i zabbix-release_latest_7.4+debian13_all.deb

![alt text](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\dpkg.png)

apt update

y ahora podemos instalar los paquetes ncesarios

apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 mariadb-server nginx

![alt text](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\install.png)

Resumiendo lo que instalamos con el comando anterior


tabla:

paquete
funcion
zabbix-server-mysql
Servidor Zabbix con soporte MySQL/MariaDB
zabbix-frontend-php
Interfaz web de Zabbix
zabbix-nginx-conf
Configuración de Nginx para Zabbix
zabbix-sql-scripts
Scripts iniciales de la base de datos
zabbix-agent2
Agente moderno de Zabbix
mariadb-server
Base de datos
nginx
Servidor web


vamos co la configuracion de la base de datos
systemctl enable --now mariadb

mariadb-secure-installation

yo elegi estas opciones

Switch to unix_socket authentication? n
Change the root password? n
Remove anonymous users? y
Disallow root login remotely? y
Remove test database? y
Reload privilege tables? y

dentro de mariadb

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'adminmariadb';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
FLUSH PRIVILEGES;
EXIT;

con este comando importamos la base de datos inicial

zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mariadb --default-character-set=utf8mb4 -uzabbix -p zabbix

ya una vez instalada nos volvemos a meter a mariadb y 

SET GLOBAL log_bin_trust_function_creators = 0;
EXIT;

configuramos el archivo

y el archivo zabbix-conf

![alt text](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\nginx-conf.png)

/etc/zabbix/nginx.conf

![alt text](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\zabbix-conf.png)

reiniciamos servicios

![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\rinicio-servicios.png)

systemctl restart zabbix-server zabbix-agent2 nginx php*-fpm
systemctl enable zabbix-server zabbix-agent2 nginx mariadb

systemctl status zabbix-server
![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\status-zabbix-server.jpg)

systemctl status zabbix-agent2
![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\status-zabbix-agent2.jpg)

systemctl status nginx
![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\status-nginx.jpg)

C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\status-nginx.jpg
systemctl status mariadb
![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\status-mariadb.jpg)


comprobar que nginx no tiene errores

nginx -t

ver la version de php-fpm
systemctl list-units --type=service | grep fpm

y ahora configuramos el firewall

apt install -y ufw

y configuramos el firewall con estos comandos

ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 10051/tcp
ufw enable
ufw status

![tete](C:\Users\Administrator\Desktop\zabbix\imagenes\conf-servidor\cambios-reglas-firewall.jpg)


Más adelante abriremos 443/tcp cuando configuremos HTTPS.

ya podemos entrar en la web

http://192.168.1.10

foto zabbix1

foto zabbix2

foto zabbix3

foto zabbix4

foto zabbix5

Usuario: Admin
Contraseña: zabbix

foto zabbix6


si algo falla estos comandos son utiles

Ver log del servidor Zabbix:
tail -f /var/log/zabbix/zabbix_server.log
Ver log de Nginx:
journalctl -u nginx -f
Ver log de MariaDB:
journalctl -u mariadb -f
Ver estado de Zabbix:
systemctl status zabbix-server
Comprobar puertos abiertos:
ss -tulpn
Comprobar si Zabbix escucha en 10051:
ss -tulpn | grep 10051
Comprobar si Nginx escucha en 80:
ss -tulpn | grep ':80'
