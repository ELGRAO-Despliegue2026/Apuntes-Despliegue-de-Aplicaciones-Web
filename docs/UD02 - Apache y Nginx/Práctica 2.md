# Práctica 2

Para realizar ésta práctica se supone que ya tendréis disponible la máquina virtual con debian mínimo de la práctica 1.

## 1. Realizar conexión mediante SSH al servidor

Instala en el servidor `openssh-server` haciendo:

```bash
sudo apt update
sudo apt install openssh-server
```

Para verificar que está funcionando haz `sudo systemctl status ssh`. Debería aparecer active(running).

!!! Note Nota
    En el caso real en el que se trabajaría con un servidor con una ip asignada dentro de la red se hallaría la ip mediante `ip a` y se conectaría mediante `ssh <usuario>@<ip>`. En éste caso, hemos establecido que se utilizará NAT, por lo que son redes privadas y el procedimiento es el que se indica a continuación.

Para conectarnos con el servidor desde nuestro equipo haremos `ssh <tu_usuario>@127.0.0.1 -p 2222`.

## 2. Configuración básica de Apache

A partir de tener la conexión con el servidor mediante ssh, haremos el resto de pasos desde nuestro local.

### 1. Instalación de Apache

Hay dos modos de instalar apache: mediante el sistema de paquetes predeterminado de Linux (apt) o descargando su código fuente, descomprimiendo el paquete en `/usr/local/src` compilándolo y instalándolo. 

Por simplicidad haremos la instalación desde apt, pero cambe destacar que la instalación a partir de su código fuente permite la configuración de la misma.

```bash
sudo apt update
sudo apt install apache2 -y
```

Verifica el estado

### 2. Html de Apache2

Una vez instalado Apache en el servidor, desde nuestro cliente podremos acceder a la web por defecto de Apache accediendo a 127.0.0.1:8080.

La web utiliza `/var/www/html` como repositorio. Inspecciona el repositorio. 

Para que todos los usuarios puedan modificar los archivos de dentro de `/var/www/html` sin utilizar `sudo` puedes dar permisos a todos los usuarios con:

```bash
sudo chmod 777 –R /var/www/html
```

!!! note **Nota**
    Recuerda que el al hacer `chmod 777` lo que hacemos es dar permisos de **Lectura (4) + Escritura (2) + Ejecución (1) = 7** al `<propietario><grupo><otros>`.

!!! warning **IMPORTANTE**
    Esto se puede hacer solo en un entorno de desarrollo. En un entorno de producción nunca asignes permisos completos a todos los usuarios.

### 3. Configuración de Apache

La configuración de apache se hace en los archivos de texto encontrados en `/etc/apache2/`.

Inspeccionando los archivos podemos ver la siguiente estructura:

```bash
-rw-r--r-- 1 root root  7178 Jun 11 14:28 apache2.conf
drwxr-xr-x 2 root root  4096 Sep 30 12:02 conf-available
drwxr-xr-x 2 root root  4096 Sep 30 12:02 conf-enabled
-rw-r--r-- 1 root root  1782 Jun 11 14:28 envvars
-rw-r--r-- 1 root root 31063 Dec  5  2025 magic
drwxr-xr-x 2 root root 12288 Sep 30 12:02 mods-available
drwxr-xr-x 2 root root  4096 Sep 30 12:02 mods-enabled
-rw-r--r-- 1 root root   274 Jun 11 14:28 ports.conf
drwxr-xr-x 2 root root  4096 Sep 30 12:02 sites-available
drwxr-xr-x 2 root root  4096 Sep 30 12:02 sites-enabled
```

#### Archivos de configuración

- **apache2.conf**: Es el archivo de configuración principal del servidor. En este archivo se gestionan las **rutas, procesos, límites y logs** del servidor, además de incluir el resto de archivos.

- **ports.conf**: Define los **puertos de red y direcciones IP** en los que el servidor escucha las peticiones entrantes (por ejemplo, en el puerto `80` escucha peticiones HTTP y en el `443` peticiones HTTPS)

- **envvars**: Contiene variables de entorno del SO que Apache utiliza al arrancar. 

- **magic**: Define reglas para que Apache determine el **tipo MIME** de un archivo analizando solo sus primeros bytes, en lugar de guiarse por su extensión.

#### Directorios available-enabled

Los directorios dentro de esta carpeta contienen parejas de funcionalidades con el sufijo `*-available` o `*-enabled` que tienen como finalidad permitir **activar o desactivar sitios web y configuraciones de Apache sin eliminar archivos de configuración**. 

El funcionamiento es sencillo, en los archivos dentro de `*-available` guardas **todos los archivos de configuración / sitio web etc.** y después en el respectivo `*-enabled` haces un enlace simbólico que apunta a los archivos de `*-available` que quieres activar. 

Los directorios que encontramos són:

- **sites-available/ y sites-enabled/**: 
    - **sites-available**: Aloja los archivos de cada **sitio web** o **VirtualHost** alojado en el servidor
    - **sites-enabled**: Contiene enlaces que apuntan a `sites-available` y activa sitios web con las herramientas `a2ensite` (Apache 2 Enable Site) y `a2dissite` (Apache 2 Disable Site).

- **mods-available/ y mods-enabled/**:
    Apache tiene una estructura modular que permite añadir funcionalidades específicas al servidor web a través de módulos. Por ejemplo, si se quiere permitir conexiones seguras mediante HTTPS y gestionar certificados TLS/SSL se tiene el módulo `mod_ssl`. Si se quiere reescribir URLs para hacerlas más amigables se tiene el módulo `mod_rewrite`. Si se quiere poder ejecutar archivos `.php` desde el servidor sin necesidad de comunicarse con un programa externo se puede habilitar `mod_php`, que permite ejecutar el código dinámico utilizando el motor interno de PHP y generando el HTML resultante que envía al usuario.
        
    De la misma manera que en `sites-*`, en **mods-available** se tienen las configuraciones de los módulos de apache y en **mods-enabled** están los enlacs simbólicos que apuntan a **mods-enabled**. La herramienta que usa es `a2enmod` y `a2dismod`.

- **conf-availble/ y conf-enabled/**:
    - **conf-available**: Guarda fragmentos de configuración general que no corresponden directamente a un módulo concreto, como reglas de seguridad globales, páginas de error personalizadas etc.
    - **conf_enabled**: Contiene enlaces simbólicos a **conf-available**. Se gestiona con `a2enconf` y `a2disconf`.

