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
      dhcp4: false
      addresses:
        - 192.168.7.11/24
      gateway4: 192.168.7.1 # IP estática del router R-N07
      nameservers:
        addresses: [192.168.1.1] # IP del servidor DNS R-N07
  version: 2
```


Primero, comprobamos la configuración de red ejecutando:

<button onclick="navigator.clipboard.writeText('sudo netplan try')">Copiar</button>


Si todo es correcto, aplicamos los cambios:

```bash
sudo netplan apply
```

---

### 1.2 Configuración del hostname

Configuramos el **hostname** del sistema con:

```bash
sudo hostnamectl set-hostname <nuevo_hostname>
```

Luego, actualizamos el archivo `/etc/hosts` para que el sistema reconozca el nuevo nombre del host.

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

---

### 1.6 Conexión SSH desde el router

Finalmente, comprobamos la conexión accediendo desde otro equipo o router:

```bash
ssh bchecker@<ip_del_servidor>
```

---
