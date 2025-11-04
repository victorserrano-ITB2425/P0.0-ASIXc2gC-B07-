# P0.0-ASIXc2gC-B07-

# P0.0 - Desplegament de la infraestructura

## Índice
- [1. Documentación del servidor de base de datos](#1-documentación-del-servidor-de-base-de-datos)
  - [1.1 Configuración del Netplan](#11-configuración-del-netplan)
  - [1.2 Configuración del hostname](#12-configuración-del-hostname)
  - [1.3 Actualización de paquetes](#13-actualización-de-paquetes)
  - [1.4 Creación del usuario bchecker](#14-creación-del-usuario-bchecker)
  - [1.5 Configuración de SSH](#15-configuración-de-ssh)
  - [1.6 Conexión SSH desde el router](#16-conexión-ssh-desde-el-router)

---

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
### 2.3Configuracion del firewall
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

Verificamos que esta activado
```bash
sudo systemctl status apache2
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
sudo passwd bchecker
```
### 3.4 Cambiamos el host y hostname

Vamos a
```bash
sudo nano /etc/hosts
sudo nano /etc/hostname
```
y ahi cambiamos el nombre de ubuntu por W-N07

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
### 4.5 Cambiamos el host y hostname

Vamos a
```bash
sudo nano /etc/hosts
sudo nano /etc/hostname
```
y ahi cambiamos el nombre de ubuntu por F-N07



