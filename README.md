# P0.0-ASIXc2gC-B07-

# P0.0 - Desplegament de la infraestructura

## Índice
- [Topologia de la red](#topologia-de-la-red)
- [1. Documentación del servidor de base de datos](#1-documentación-del-servidor-de-base-de-datos)
  - [1.1 Configuración del Netplan](#11-configuración-del-netplan)
  - [1.2 Configuración del hostname](#12-configuración-del-hostname)
  - [1.3 Actualización de paquetes](#13-actualización-de-paquetes)
  - [1.4 Creación del usuario bchecker](#14-creación-del-usuario-bchecker)
  - [1.5 Configuración de SSH](#15-configuración-de-ssh)
  - [1.6 Conexión SSH desde el router](#16-conexión-ssh-desde-el-router)
  - [1.7. Instalación del paquete mysql-server](#17-instalamos-el-paquete-de-mysql-server)
  - [1.8. Verificación del estado de mysql](#18-verificamos-el-estado-del-mysql)
  - [1.9. Securización de mysql](#19-securisamos-el-mysql-con-el-script)
  - [1.10. Proceso de creación y análisis de datos](#110-proceso-de-creación-y-análisis-de-datos)
    - [Creación de la estructura de datos](#creación-de-la-estructura-de-datos)
    - [Descarga y análisis del CSV](#descarga-y-análisis-del-csv)
    - [Creación de la tabla equipaments](#creación-de-la-tabla-equipaments)
  - [1.11. Creación de usuario de aplicación y seguridad](#111-creación-de-usuario-de-aplicación-y-seguridad) 
- [2. Documentacion de la configuracion del router](#2-documentacion-de-la-configuracion-del-router)
  - [2.1 Configuración del host y hostname](#21-configuración-del-host-y-hostname)
  - [2.2 Configuramos el Netplan](#22-configuramos-el-netplan)
  - [2.3 Configuracion del firewall](#23-Configuracion-del-firewall)
  - [2.4 Instalación del DHCP](#24-Instalación-del-DHCP)
  - [2.5 Configuracion del DHCP](#25-Configuracion-del-DHCP)
  - [2.6 Creación del usuario bchecker](#26-Creación-del-usuario-bchecker)
  - [2.7 Configuración del DNS](#27-Configuración-del-DNS)
- [3. Documentación de apache 2](#3-documentación-de-apache-2)
  - [3.1 Actualización del servidor e instalación de apache](#31-Actualización-del-servidor-e-instalación-de-apache)
  - [3.2 Configuración del Netplan](#32-Configuración-del-netplan)
  - [3.3 Creación del usuario bchecker](#33-creacion-del-usuario-bchecker)
  - [3.4 Cambiamos el host y hostname](#34-cambiamos-el-host-y-hostname)
  - [3.5 Configuración html, php, js y css](#35-configuración-html,-php,-js-y-css)
- [4. Documentación de ftpserver](#4-documentación-de-ftpserver)
  - [4.1 Instalacion de servicio FTP](#41-instalación-de-servicio-ftp)
  - [4.2 Configuración FTP](#42-Configuración-ftp)
  - [4.3 Configuración del Netplan](#43-configuración-del-netplan)
  - [4.4 Creación del usuario bchecker](#44-creación-del-usuario-bchecker)
  - [4.5 Cambiamos el host y hostname](#45-cambiamos-el-host-y-hostname)
- [5 Configuracion de los clientes](#5-Configuracion-de-los-clientes)
  - [5.1 Linux](#51-linux)
  - [5.2 Windows](#52-Windows)
---
## Topologia de la red
![TOPOLOGIA](https://github.com/victorserrano-ITB2425/P0.0-ASIXc2gC-B07-/blob/e635d16e8e20bb264951e443377a07ca827af339/img/TOPOLOGIA_G07.png)
## 1. Documentación del servidor de base de datos

### 1.1 Configuración del Netplan
Editamos el archivo de configuración de red:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
Configuracion del netplan:
```bash
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp3s0:
      dhcp4: true # la ip la tendremos mediante el Router
  version: 2
```

Primero, comprobamos la configuración de red ejecutando:

```bash
sudo netplan try
```

Si todo es correcto, aplicamos los cambios:

```bash
sudo netplan apply
```
Pedimos la ip al R-N07:
```bash
sudo dhclient -v
```
Resultado:

```bash
Internet Systems Consortium DHCP Client 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp3s0/52:54:00:07:6a:7c
Sending on   LPF/enp3s0/52:54:00:07:6a:7c
Listening on LPF/enp2s0/52:54:00:4b:44:2a
Sending on   LPF/enp2s0/52:54:00:4b:44:2a
Listening on LPF/enp1s0/52:54:00:2d:54:24
Sending on   LPF/enp1s0/52:54:00:2d:54:24
Sending on   Socket/fallback
DHCPDISCOVER on enp3s0 to 255.255.255.255 port 67 interval 3 (xid=0x381aa044)
DHCPDISCOVER on enp2s0 to 255.255.255.255 port 67 interval 3 (xid=0xd7f73f39)
DHCPDISCOVER on enp1s0 to 255.255.255.255 port 67 interval 3 (xid=0x2cad3735)
DHCPOFFER of 192.168.7.12 from 192.168.7.1
DHCPREQUEST for 192.168.7.12 on enp3s0 to 255.255.255.255 port 67 (xid=0x44a01a38)
DHCPACK of 192.168.7.12 from 192.168.7.1 (xid=0x381aa044)
bound to 192.168.7.12 -- renewal in 250 seconds.
```
Revisamos la ip del servidor B-N07:
```bash
isard@B-N07:~$ ip a | grep enp3s0
```
Resultado esperado:
```bash
4: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.7.12/24 brd 192.168.7.255 scope global dynamic enp3s0
```

---

### 1.2 Configuración del hostname

Configuramos el **hostname** del sistema con:

```bash
sudo hostnamectl set-hostname B-N07
```

Luego, actualizamos el archivo `/etc/hosts` para que el sistema reconozca el nuevo nombre del host.
```bash
sudo nano /etc/hosts
```
Tendriamos que tener algo asi:
```bash
127.0.0.1 localhost
127.0.1.1 B-N07
```

---

### 1.3 Actualización de paquetes

Actualizamos los paquetes del sistema para asegurarnos de tener las últimas versiones:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 1.4 Creación del usuario bchecker

Creamos el usuario `bchecker` con la contraseña `bchecker121` para permitir el acceso seguro mediante SSH:

```bash
sudo adduser bchecker
sudo passwd bchecker
```

Le damos privilegios de **sudo**:

```bash
sudo usermod -aG sudo bchecker
```

---

### 1.5 Configuración de SSH

Editamos el archivo de configuración de SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Buscamos la línea que permite el acceso con root y la desactivamos por seguridad:

```
PermitRootLogin no
```

Reiniciamos el servicio SSH para aplicar los cambios:

```bash
sudo systemctl restart ssh
```
Comprobamos el estado del ssh
```bash
sudo systemctl status ssh
```


---

### 1.6 Conexión SSH desde el router

Finalmente, comprobamos la conexión accediendo desde otro equipo o router:

```bash
ssh bchecker@192.168.7.11
```

Desde el otro equipo nos deve aparecer esto:
```bash
isard@R-N07:~$ ssh bchecker@192.168.7.11
The authenticity of host '192.168.7.11 (192.168.7.11)' can't be established.
ED25519 key fingerprint is SHA256:IP3d7VGhqa1/1N6jLfFMJaUL9jwki+MYqxzLz67moiw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.7.11' (ED25519) to the list of known hosts.
bchecker@192.168.7.11's password:
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-160-generic x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

bchecker@B-N07:~$
```
### 1.7 Instalamos el paquete de mysql-server:
```bash
sudo apt install mysql-server
```
### 1.8 Verificamos el estado del mysql:
```bash
sudo systemctl status mysql
```
Resultado esperado: 
```
lsard@B-N07:~$ sudo systemctl status mysql
mysql.service - MySQL Community Server
Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2025-10-28 15:29:16 CET; 40s ago
Process: 1752 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
Main PID: 1779 (mysqld)
Status: "Server is operational"
Tasks: 38 (limit: 4554)
Memory: 364.3M
CPU: 1.327s
CGroup: /system.slice/mysql.service
        └─1779 /usr/sbin/mysqld

Oct 28 15:29:15 B-N07 systemd[1]: Starting MySQL Community Server...
Oct 28 15:29:16 B-N07 systemd[1]: Started MySQL Community Server.
lsard@B-N07:~$ $
```
### 1.9 Securisamos el mysql con el script:
Validate Password Component?: N (No). ya que la contraseña que nos piden como requisito es devil pero lo ideal sería activarla:
```bash
lsard@B-N07:~$ sudo mysql_secure_installation
```
Resultado:
```
lsard@B-N07:~$ sudo mysql_secure_installation

Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: n
```
Remove anonymous users?: Y (Sí) para borrar usuarios que no necesitan contraseña para acceder:

```
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.

Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.
```
Remove test database and access to it?: Y (Sí) Elimina una base de datos de prueba (test) que es un riesgo de seguridad:

```
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
- Dropping test database...
Success.

- Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.
```
Reload privilege tables now?: Y (Sí). Es como "Guardar y Aplicar" todos los cambios anteriores.


```
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
lsard@B-N07:~$
```
### 1.10 Proceso de creación y análisis de datos
Accede a la consola de MySQL:
```bash
sudo mysql -u root -p
```
Resultado:
```
lsard@B-N07:~$ sudo mysql -u root -p
Enter password:
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 8.0.43-0ubuntu0.22.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
#### Creación de la estructura de datos
Creamos la base de datos
```bash
CREATE DATABASE bcn_educacion;
```
Resultado:
```
mysql> CREATE DATABASE bcn_educacion;
Query OK, 1 row affected (0.01 sec)
```
#### Descarga y análisis del CSV
Extraemos el archivo csv al servidor para poder analizarlo:
```bash 
lsard@B-N07:~$ wget -O equipaments.csv "https://opendata-ajuntament.barcelona.cat/data/dataset/f36b60f2-9541-4d08-b0f9-b0a9313fab3d/resource/29d9ff10-6892-4f16-9012-d5c4997857e7/download"
```
Resultado:
```
lsard@B-N07:~$ wget -O equipaments.csv "https://opendata-ajuntament.barcelona.cat/data/dataset/f36b60f2-9541-4d08-b0f9-b0a9313fab3d/resource/29d9ff10-6892-4f16-9012-d5c4997857e7/download"
--2025-10-28 15:50:31--  https://opendata-ajuntament.barcelona.cat/data/dataset/f36b60f2-9541-4d08-b0f9-b0a9313fab3d/resource/29d9ff10-6892-4f16-9012-d5c4997857e7/download
Resolviendo opendata-ajuntament.barcelona.cat (opendata-ajuntament.barcelona.cat)... 52.138.192.38
Conectando con opendata-ajuntament.barcelona.cat (opendata-ajuntament.barcelona.cat)|52.138.192.38|:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 2220488 (2,1M) [text/csv]
Guardando como: 'equipaments.csv'

equipaments.csv     100%[==========================================>]  2,12M  5,45MB/s    en 0,4s

2025-10-28 15:50:33 (5,45 MB/s) - 'equipaments.csv' guardado [2220488/2220488]

lsard@B-N07:~$ ls -l equipaments.csv
-rw-rw-r-- 1 lsard lsard 2220488 oct 28 15:50 equipaments.csv
```
Extraemos la primera fila para saber los nombres de las cabeceras:
```bash
head -n 1 equipaments.csv

```
Resultdo:
```
isard@B-N07:~$head -n 1 equipaments.csv
��register_id,name,institution_id,institution_name,created,modified,addresses_roadtype_id,addresses_roadtype_name,addresses_road_id,addresses_road_name,addresses_start_street_number,addresses_end_street_number,addresses_neighborhood_id,addresses_neighborhood_name,addresses_district_id,addresses_district_name,addresses_zip_code,addresses_town,addresses_main_address,addresses_type,values_id,values_attribute_id,values_category,values_attribute_name,values_value,values_outstanding,values_description,secondary_filters_id,secondary_filters_name,secondary_filters_fullpath,secondary_filters_tree,secondary_filters_asia_id,geo_epgs_25831_x,geo_epgs_25831_y,geo_epgs_4326_lat,geo_epgs_4326_lon,estimated_dates,start_date,end_date,timetable
isard@B-N07:~$ 

```
#### Creación de la tabla equipaments

Entramos a la base de datos para crear las tablas:
```bash
USE bcn_educacion;
```
Resultado:
```
mysql> USE bcn_educacion;
Database changed
```
Analizamos el csv y seleccionamos las columnas más útiles y liamos un poco los nombres:
```bash
CREATE TABLE equipaments (
-> id INT AUTO_INCREMENT PRIMARY KEY,
-> nom_equipament VARCHAR(255),
-> adreca VARCHAR(255),
-> barri VARCHAR(100),
-> districte VARCHAR(100),
-> codi_postal VARCHAR(10),
-> latitud DECIMAL(10, 8),
-> longitud DECIMAL(11, 8)
-> );
```
Resultado:  
```
mysql> CREATE TABLE equipaments (
-> id INT AUTO_INCREMENT PRIMARY KEY,
-> nom_equipament VARCHAR(255),
-> adreca VARCHAR(255),
-> barri VARCHAR(100),
-> districte VARCHAR(100),
-> codi_postal VARCHAR(10),
-> latitud DECIMAL(10, 8),
-> longitud DECIMAL(11, 8)
-> );
Query OK, 0 rows affected (0.05 sec)
mysql>
```
Preparación y Carga de Datos (ETL)

Movemos el archivo CSV a la carpeta segura de MySQL:
```bash
sudo cp equipaments.csv /var/lib/mysql-files/equipaments.csv
```
Ajustamos la codificación a UTF-8:
```bash
sudo iconv -f UTF-16LE -t UTF-8 /var/lib/mysql-files/equipaments.csv -o /var/lib/mysql-files/equipaments_utf8.csv
```
Creamos el script limpiar_datos.py  para procesar el CSV y formatear los nulos:
```bash
sudo nano limpiar_datos.py && sudo chmod +x limpiar_datos.py 
```

```bash
import csv

input_file = '/var/lib/mysql-files/equipaments_utf8.csv'
output_file = '/var/lib/mysql-files/equipaments_final.csv'

with open(input_file, 'r', encoding='utf-8') as fin, \
     open(output_file, 'w', encoding='utf-8', newline='') as fout:
    
    reader = csv.reader(fin)
    writer = csv.writer(fout, lineterminator='\n')
    
    next(reader) # Salta cabecera
    
    for row in reader:
        try:
            nombre = row[1]
            direccion = row[9] + " " + row[10]
            barri = row[13]
            districte = row[16]
            cp = row[15]
            lat = row[34] if row[34] else '\\N'
            lon = row[35] if row[35] else '\\N'
            
            writer.writerow([nombre, direccion, barri, districte, cp, lat, lon])
        except IndexError:
            continue

print("Archivo limpio creado: equipaments_final.csv")
```
Ejecutamos el script:
```bash
sudo python3 limpiar_datos.py
```
Podemos ver el resultado para comprobar que el texto esta limpio:
```bash
sudo cat /var/lib/mysql-files/equipaments_final.csv
```
'''
isard@B-N07:~$ sudo cat /var/lib/mysql-files/equipaments_final.csv
Escola Bou Centelles,LluIl 198,el Poblenou,Sant Marti,08005,41.401178206266216,2.202556982064
0296
Autoescola Canyelles *Antonio Machado,Antonio Machado 16,Canyelles,Nou Barris,08042,41.443458
12450906,2.168307205987914
Escola de Música Farré,C Santa Caterina 33,Sants,Sants-Montjuic,08014,41.37927459385171,2.136
181343415099
Escola Professional de Pràctica Jurídica,Mallorca 283,la Dreta de l'Eixample,Eixample,08037,4
1.39565882210395,2.164768994816127
Llar d'Infants Dintell,Balmes 251,Sant Gervasi - Galvany,Sarrià-Sant Gervasi,08006,41.3998672
71542966,2.1493835109212776
Escola Bressol Municipal Cadí,Carrer de l'Om 11,el Raval,Ciutat Vella,08001,41.37613418778239
,2.1726426208041967
Escola Bressol Municipal Cadí,Carrer de l'Om 11,el Raval,Ciutat Vella,08001,41.37613418778239
,2.1726426208041967
Institut d'Estudis Catalans,C Carme 47,el Raval,Ciutat Vella,08001,41.38152411821448,2.168974
973139409
Escola Massana Centre Municipal d'Art i Disseny,Plaça de la Gardunya 9,el Raval,Ciutat Vella,
08001,41.380953983657726,2.1700553338176225
Facultat de Nàutica - UPC,Pla Palau 18,la Barceloneta,Ciutat Vella,08003,41.38243904810675,2.
184497630278131
Autoescola Pallars *Rambla Brasil,Rbla Brasil 1,Sants - Badal,Sants-Montjuïc,08028,41.3757903
'''

Entramos a la base de datos y argamos los datos en MySQL:
```bash
sudo mysql -u root -p
```

Ejecutamos:
```bash
LOAD DATA INFILE '/var/lib/mysql-files/equipaments_final.csv'
INTO TABLE equipaments
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
(nom_equipament, adreca, barri, districte, codi_postal, latitud, longitud);
```
Comprobamos el contenido de las tablas:

```bash
SELECT * FROM equipaments LIMIT 5;
```
```
mysql> SELECT * FROM equipaments LIMIT 5;
+----+---------------------------------------+--------------------+-----------------------+
| id | nom_equipament                        | adreca             | barri                 
     | districte          | codi_postal | latitud    | longitud   |
+----+---------------------------------------+--------------------+-----------------------+
|  1 | Escola Bou Centelles                  | Llull 198          | el Poblenou           
     | Sant Martí         | 08005       | 41.40117821 | 2.20255698 |
|  2 | Autoescola Canyelles *Antonio Machado | Antonio Machado 16 | Canyelles             
     | Nou Barris         | 08042       | 41.44345812 | 2.16830721 |
|  3 | Escola de Música Farré                | C Santa Caterina 33| Sants                 
     | Sants-Montjuïc     | 08014       | 41.37927459 | 2.13618134 |
|  4 | Escola Professional de Pràctica Jurídica     | Mallorca 283       | la Dreta de l'Eixam
ple  | Eixample           | 08037       | 41.39565882 | 2.16476899 |
|  5 | Llar d'Infants Dintell                | Balmes 251         | Sant Gervasi - Galv   
any  | Sarrià-Sant Gervasi| 08006       | 41.39986727 | 2.14938351 |
+----+---------------------------------------+--------------------+-----------------------+
5 rows in set (0.00 sec)
```

### 1.11 Creación de usuario de aplicación y seguridad
Modificamos el archivo /etc/mysql/mysql.conf.d/mysqld.cnf para permitir conexiones externas:
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Modificamos la linea bind-address colocando la red 0.0.0.0 para indicarle a la web que reciba las llamadas de cualquier red, mas adelante configuraremos una regla en el firewall para que no sea inseguro:
```bash
bind-address = 0.0.0.0
```
Reiniciamos el servicio MySQL:
```bash
sudo systemctl restart mysql &&  sudo systemctl status mysql
```
Entramos a la Base De Datos y creamos el usuario específico para la aplicación web con restricción de IP:
```bash
sudo mysql -u root -p
```
```bash
CREATE USER 'app_bcn'@'192.168.17.11' IDENTIFIED BY 'pirineus';
GRANT SELECT, INSERT, UPDATE, DELETE ON bcn_educacion.* TO 'app_bcn'@'192.168.17.11';
FLUSH PRIVILEGES;
```
Configuramos el Firewall (UFW) para proteger el puerto 3306:
```bash
sudo apt install ufw -y
sudo ufw allow from 192.168.17.11 to any port 3306 proto tcp
sudo ufw allow ssh
sudo ufw enable

```
Comprobamos el estado:
```bash
sudo ufw status
```
Comprovamos el funcionamiento desde el servidor WEB:
```bash
mysql -u app_bcn -p -h 192.168.7.11
```
Resultado:
```
isard@W-N07:~$ mysql -u app_bcn -p -h 192.168.7.11
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.43-0ubuntu0.22.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## 2. Documentacion de la configuracion del router
### 2.1 Configuración del host y hostname
Configuramos el archivo de hostname: 
```bash
sudo hostnamectl set-hostname R-N07
```
Tambien editamos el archivo de hosts:
```bash
sudo nano /etc/hosts
```
Tendriamos que tener algo asi:
```bash
127.0.0.1 localhost
127.0.1.1 hostname
```
### 2.2 Configuramos el Netplan
Entramos al archivo de netplan:
```bash
sudo nano /etc/netplan/00-installer-config.yaml 
```
Configuracion del  netplan
```bash
network:
  version. 2
  ethernets:
     enp1s0:
       dhcp4: true #nat
      enp3s0:
        addresses:
          - 192.1668.7.1/24 #Intranet B-N07
      enp4s0:
        addresses:
          - 192.168.17.1/24 #DMZ
```
Aplicamos los cambios:
```bash
sudo netplan apply
```
### 2.3 Configuracion del firewall
Configuramos el firewall para que nuestra red interna compata una unica direccion IP publica al acceder a internet
```bash
sudo iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
```
### 2.4 Instalación del DHCP
Instalamos el DHCP con el siguiente comando:
```bash
sudo apt install isc-dhcp-server
```
### 2.5 Configuracion del DHCP
Entramos al archivo de configuracion de interfaces de red:
```bash
sudo nano /etc/default/isc-dhcp-server
```
Al entrar buscaremos la linea de las interfaces y le indicamos al servidor DHCP exactamente en que interfaces queremos el DHCP
```bash
INTERFACESv4="enp3s0 enp4s0"
INTERFACESv6=""
```
Entramos el fichero  de configuracion de DHCP:
```bash
sudo nano /etc/dhcp/dhcpd.conf
```
Al entrar al archivo de configuracion de DHCP añadimos el rango de Ips:
```bash
#Subnet 7.1 Intranet
subnet 192.168.7.0 netmask 255.255.255.0 {
  range 192.168.7.10 192.168.7.100;
  option routers 192.168.7.1;
  option subnet-mask 255.255.255.255.0
  option domain-name-servers 192.168.17.1;
}
#Subnet 17.0.DMZ
subnet 192.168.17.0 netmask 255.255.255.0 {
  range 12.168.17.10.192.168.17.100;
  option routes 192.168.17.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 192.168.17.1
}
```
En el mismo fichero de configuracion de DHCP colocamos una ip fija a los servidores,mediantes la direccion MAC:
```bash
#Intranet 7.1
#Intranet 7.1
host B-N07 {
    hardware ethernet 52:54:00:07:6c:7c;
    fixed-address 192.168.7.11;
}

#DMZ 17.1
host W-N07 {
    hardware ethernet 52:54:00:09:44:86;
    fixed-address 192.168.17.11;
}

#F-N07
host F-N07 {
    hardware ethernet 52:54:00:34:5e:8b;
    fixed-address 192.168.17.12;
}
```
### 2.6 Creación del usuario bchecker

Creamos otro  usuario `bchecker` en el servidor del router con la contraseña `bchecker121` para permitir el acceso seguro mediante SSH:

```bash
sudo adduser bchecker
sudo passwd bchecker
```

Le damos privilegios de **sudo**:

```bash
sudo usermod -aG sudo bchecker
```
### 2.7 Configuración del DNS

Instalamos el paquete bind9 que es el encargado de que se hará de DNS, con esto resolveremos los dominios como w-n07.dmz.local etc
```bash
 sudo apt install bind9 bind9utils bind9-doc -y
```
Después de la instalación, verificamos que el servicio esté activo y corriendo sin problemas
```bash
 sudo systemctl status bind9.service
sudo systemctl status apache2
named.service - BIND Domain Name Server
  Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: ena
  Active: active (running) since Tue 2025-10-28 16:25:11 CET; 39s ago
  Docs: man:named(8)
  Main PID: 1508 (named)
  Tasks: 14 (limit: 4554)
  Memory: 24.4M
  CPU: 111ms
  CGroup: /system.slice/named.service
          └─1508 /usr/sbin/named -u bind
```
En esta imagen configuramos la red, del dns, con recursion yes, lo que hacemos es permitir que nuestro servidor DNS, realice las búsquedas recursivas, esto quiere decir que pregunta en otros servidores o equipos en caso de que no sea capaz de resolver
Por otro lado en allow query le indicamos que redes están autorizadas, para realizar consultas en nuestro servidor DNS
```bash
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        recursion yes;

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        allow-query { 192.168.7.0/24; 192.168.17.0/24; };

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on { 192.168.7.1; 192.168.17.1; 127.0.0.1; };

        listen-on-v6 { none; };
};
```
En esta captura reiniciamos y el servicio y lo verificamos, además de que ejecutamos named-checkconf para comprobar los archivos de configuración que hemos creado no tenga ningún tipo de error
```bash
sudo named-checkconf
sudo systemctl restart bind9
sudo systemctl status bind9
named. service - BIND Domain Name Server
  Loaded: loaded (/1ib/systemd/system/named.service; enabled; vendor preset: enabled)
  Active: active (running) since Tue 2025-10-28 16:38:11 CET; 4min 43s ago
  Docs: man: named (8)
  Process: 1637 ExecStart=/usr/sbin/named OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 1639 (named)
    Tasks: 14 (limit: 4554)
  Memory: 23.6M
    CPU: 104ms
    CGroup: /system.slice/named.service
              -1639 /usr/sbin/named -u bind
Oct 28 16:38:11 R-N07 named [1639]: configuring command channel from'/etc/bind/rndc.key'
Oct 28 16:38:11 R-N07 named [1639]: command channel listening on :: 1#953
Oct 28 16:38:11 R-N07 named [1639]: managed-keys-zone: loaded serial 2
Oct 28 16:38:11 R-N07 named [1639]:zone localhost/IN: loaded serial 2
Oct 28 16:38:11 R-N07 named [1639]:zone 0. in-addr.arpa/IN: loaded serial 1
Oct 28 16:38:11 R-N07 named [1639]:zone 255.in-addr.arpa/IN: loaded serial 1
Oct 28 16:38:11 R-N07 named [1639]:zone 127.in-addr.arpa/IN: loaded serial 1
Oct 28 16:38:11 R-N07 named [1639]:all zones loaded
Oct 28 16:38:11 R-N07 named [1639]:running
Oct 28 16:38:11 R-N07 named [1639]: Started BIND Domain Name Server.
```
En esta captura dispondremos de un archivo de configuración donde definimos las zonas DNS, que lo usara y gestionará el propio DNS 
```bash
//
// Do any local configuration here
//organization
//include "/etc/bind/zones.rfc1918";
// Zona Forward para Intranet
zone "intranetg07. lan" {
    type master; 
    file "/etc/bind/db.intranetg07.lan";
};
// Zona Forward para DMZ
zone "dmzg07.lan" {
    type master;
    file "/etc/bind/db.dmzg07. 1an";
};
// Zona Reverse para Intranet (192.168.7.x)
zone "7.168.192. in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.7";
};
// Zona Reverse para DMZ (192.168.17.X)
zone "17.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.17";
};
```
Este archivo contiene los registros de cada equipo del dominio g07. Por ejemplo:
```bash
sudo named-checkconf
sudo named-ckeckzone intranetg07.lan /etc/bind/db.intranetg07.lan
Zone intranetg07.lan/IN: loaded serial 3
OK
sudo named-ckeckzone dmzg07.lan /etc/bind/db.dmzg07.lan
Zone dmzg07.lan/IN: loaded serial 3
OK
sudo named-chekzone 7.168.192.in-addr.arpa /etc/bind/db.192.168.7
zone 7.168.192.in-addr.arpa/IN: loaded serial 2
OK
sudo named-checkzone 17.168.192.in-addr.arpa /etc/bind/db.192.168.17
zone 17.168.192.in-addr.arpa/IN: loaded serial 2
OK
```
## 3. Documentación de apache 2

### 3.1 Actualización del servidor e instalación de apache
Actualizamos el servidor
```bash
sudo apt update
sudo apt upgrade -y
```

Instalamos el servicio de apache 
```bash
sudo apt install apache2 -y
```

### 3.2 Configuración del Netplan


Editamos el archivo de configuración de red:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuracion del netplan:
```bash
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp3s0:
      dhcp4: true # la ip la tendremos mediante el Router
  version: 2
```

Primero, comprobamos la configuración de red ejecutando:

```bash
sudo netplan try
```

Si todo es correcto, aplicamos los cambios:

```bash
sudo netplan apply
```
Ahora ya tenemos una IP asignada por el router mediante dhcp (ver con ip a)

### 3.3 Creación del usuario bchecker

Creamos otro  usuario `bchecker` con la contraseña `bchecker121` para permitir el acceso seguro mediante SSH:

```bash
sudo adduser bchecker
sudo passwd bchecker121
```
### 3.4 Cambiamos el host y hostname

Vamos a
```bash
sudo nano /etc/hosts
sudo nano /etc/hostname
```
y ahi cambiamos el nombre de ubuntu por W-N07

### 3.5 Configuración html, php, js y css

Pondremos todos los archivos necesarios para la pagina web en /var/www/html/mi_sitio/public_html

<img width="558" height="48" alt="image" src="https://github.com/user-attachments/assets/c942f768-d84b-4d52-ae44-49b9f4d33df3" />

En index.html tendremos la pagina principal, a la cual accederemos con la IP 192.168.17.11 y mediante un boton accederemos a taula.php para ver la tabla que esta en el servidor BBDD

El estilo de las paginas y los botones estan hechos con el css y el js respectivamente.

## 4. Documentación de ftpserver

### 4.1 Instalacion de servicio FTP
El servicio se instala con los siguientes comandos
```bash
sudo apt update
sudo apt install vsftpd -y
```

Vemos que el servicio funciona con
```bash
sudo systemctl status vsftpd
```
### 4.2 Configuración FTP
Vamos a cambiar los parametros 
"anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES"
que se encuentran en 
```bash
sudo nano /etc/vsftpd.conf
```
y reiniciamos el servicio para aplicar los cambios
```bash
sudo systemctl restart vsftpd
```

### 4.3 Configuración del Netplan
Editamos el archivo de configuración de red:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuracion del netplan:
```bash
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp3s0:
      dhcp4: true # la ip la tendremos mediante el Router
  version: 2
```

Primero, comprobamos la configuración de red ejecutando:

```bash
sudo netplan try
```

Si todo es correcto, aplicamos los cambios:

```bash
sudo netplan apply
```
Ahora ya tenemos una IP asignada por el router mediante dhcp (ver con ip a)

### 4.4 Creación del usuario bchecker

Creamos otro  usuario `bchecker`con la contraseña `bchecker121` para permitir el acceso seguro mediante SSH:

```bash
sudo adduser bchecker
sudo passwd bchecker
```
y creamos una carpeta con los permisos adecuados para que suba las imagenes con
```bash
sudo mkdir /home/bchecker/uploads
sudo chmod 775 /home/bchecker/uploads
sudo chown bchecker:bchecker /home/bchecker/uploads
```
y volvemos a reiniciar el servicio vsftpd como antes

### 4.5 Cambiamos el host y hostname

Vamos a
```bash
sudo nano /etc/hosts
sudo nano /etc/hostname
```
y ponemos el host F-N07
### 5 Configuracion de los clientes
### 5.1 Linux
Le configuramos las ip de forma estatica y le colocamos la siguiente ip 
```bash
ip a
```
```bash
enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:7d:b6:a8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.7.14/24 brd 192.168.7.255 scope global enp3s0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe7d:b6a8/64 scope link 
       valid_lft forever preferred_lft forever
```
Y Comprobamos que hace ping desde el cliente ubuntu a router 
```bash
isard@ubuntu:~$ ping 192.168.17.1
```
Resultado:
```bash
isard@ubuntu:~$ ping 192.168.17.1
PING 192.168.17.1 (192.168.17.1) 56(84) bytes of data.
64 bytes from 192.168.17.1: icmp_seq=1 ttl=64 time=4.53 ms
64 bytes from 192.168.17.1: icmp_seq=2 ttl=64 time=1.90 ms
```
### 5.2 Windows
Cambiamos la ip de forma estatica y colocamos la siguiente:
```bash
ipconfig /all
```
```bash
adaptador de Ethernet Ethernet:
Sufijo DNS específico para la conexión....:
Descripcion...............................:Red Hat VirtIO Ethernet Adapter #3
Dirección física...........................:52-54-00-48-1A-04
DHCP habilitado............................:no
Configuración automática habilitada........:si
Dirección IPv4.............................:192.168.7.69(Preferido)
Máscara de subred .........................:255.255.255.0
Puerta de enlace predeterminada:...........:192.168.7.1
Servidores DNS.............................:192.168.7.1
NetBIOS sobre TCP/IP.......................:habilitado
```
Aqui tendremos las distintas comprobaciones, con el comando nslookup tanto la del Web y la del FTP por otro lado tenemos los ping de cada uno de los servidores
```bash
C:\Windows\system32> nslookup
```
Resultado de la comprobaciones:
```bash
servidores
PS C: \WINDOWS\system32> nslookup W-N07.dmzg07.1lan
Servidor:R-N07.intranetg07.lan
Address: 192.168.7.1
Nombre: W-N07.dmzg07.lan
Address: 192.168.17.11

PS C: \WINDOWS\system32> nslookup F-N07.dmzg07.lan
Servidor: R-N07.intranetg07.lan
Address: 192.168.7.1
Nombre:F-N07.dmzg07.lan
Address: 192.168.17.12

PS C: \WINDOWS\system32> PING F-N07.dmzg07.lan
Haciendo ping a F-N07.dmzg07.lan [192.168.17.12] con 32 bytes de datos:
Respuesta desde 192.168.17.12: bytes=32 tiempo=2ms TTL=63
Respuesta desde 192.168.17.12: bytes=32 tiempo=1ms TTL=63
Estadísticas de ping para 192.168.17.12:
    Paquetes: enviados = 2, recibidos = 2, perdidos = 0
    (0% perdidos),
Tiempos aproximados de ida y vuelta en milisegundos:
    Mínimo = 1ms, Máximo = 2ms, Media = 1ms

PS C: \WINDOWS\system32> ping F-N07.dmzg07.lan
Haciendo ping a F-N07.dmzg07.lan [192.168.17.12] con 32 bytes de datos:
Respuesta desde 192.168.17.12: bytes=32 tiempo=2ms TTL=63
Respuesta desde 192.168.17.12: bytes=32 tiempo=1ms TTL=63
Estadísticas de ping para 192.168.17.12:
Paquetes: enviados = 2, recibidos = 2, perdidos = 0
Tiempos aproximados de ida y vuelta en milisegundos:
    Mínimo = 1ms, Máximo = 2ms, Media = 1ms
```



