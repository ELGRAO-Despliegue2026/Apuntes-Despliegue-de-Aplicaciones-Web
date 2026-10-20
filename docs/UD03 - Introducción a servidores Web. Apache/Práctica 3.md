<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD02 - Apache y Nginx/Práctica 3.md -->
# Práctica 3

En esta práctica desplegaremos **dos sitios web** en el mismo servidor Apache, diferenciados mediante **VirtualHost por nombre de dominio (Hostname)**.

Se supone que ya tienes disponible la máquina virtual con Apache instalado de las prácticas anteriores.

!!! warning Contexto del laboratorio
    Los ordenadores del laboratorio son equipos **Linux Mint sin permisos de administrador**. Por ello, en lugar de modificar el fichero `/etc/hosts` de tu equipo anfitrión (lo cual requiere `sudo`/permisos de administrador), usaremos el servicio **nip.io**, que resuelve automáticamente nombres de dominio a direcciones IP sin necesidad de tocar ningún fichero del sistema.

## 1. ¿Qué es nip.io?

[nip.io](https://nip.io) es un servicio DNS gratuito que permite resolver cualquier dirección IP a un nombre de dominio **sin configuración adicional**, usando el siguiente formato:

```text
<cualquier-texto>.<IP>.nip.io
```

Por ejemplo, si la IP de tu máquina virtual es `192.168.1.50`, entonces `miweb1.192.168.1.50.nip.io` y `miweb2.192.168.1.50.nip.io` se resolverán automáticamente a esa misma IP.

De esta forma podemos simular dos dominios distintos (`miweb1...` y `miweb2...`) que apuntan al mismo servidor, sin necesidad de permisos de administrador ni de modificar ficheros de resolución DNS.

## 2. Averigua la IP de tu máquina virtual

Conéctate por SSH a tu máquina virtual (como en la práctica 2) y ejecuta:

```bash
ip a
```

Anota la dirección IP de la interfaz de red de la máquina virtual (por ejemplo `192.168.1.50`). La llamaremos `<IP>` durante el resto de la práctica.


A partir de ahora, los dos dominios que usaremos serán:

- `miweb1.<IP>.nip.io`
- `miweb2.<IP>.nip.io`

!!! example Ejemplo
    Si `<IP>` es `127.0.0.1`, los dominios serán `miweb1.127.0.0.1.nip.io` y `miweb2.127.0.0.1.nip.io`.

Comprueba que la resolución funciona haciendo ping (sustituyendo `<IP>` por tu IP real):

```bash
ping -c 3 miweb1.<IP>.nip.io
```

Deberías ver que el dominio se resuelve a tu `<IP>`.

## 3. Crea las carpetas y los contenidos de cada web

Crearemos dos directorios, uno para cada sitio web, dentro de `/var/www/`.

```bash
sudo mkdir -p /var/www/miweb1
sudo mkdir -p /var/www/miweb2
```

### Contenido de miweb1 (con `index.html`)

```bash
sudo nano /var/www/miweb1/index.html
```

```html
<!DOCTYPE html>
<html>
<head><title>Mi Web 1</title></head>
<body>
    <h1>Bienvenido a Mi Web 1</h1>
    <p>Esta web utiliza index.html como página de inicio.</p>
</body>
</html>
```

### Contenido de miweb2 (con `miweb.html`)

En este caso, la página de inicio **no se llamará `index.html`**, sino `miweb.html`, por lo que más adelante tendremos que indicarle a Apache cuál es el archivo de índice mediante `DirectoryIndex`.

```bash
sudo nano /var/www/miweb2/miweb.html
```

```html
<!DOCTYPE html>
<html>
<head><title>Mi Web 2</title></head>
<body>
    <h1>Bienvenido a Mi Web 2</h1>
    <p>Esta web utiliza miweb.html como página de inicio.</p>
</body>
</html>
```

Asigna permisos para poder trabajar sin `sudo` en estas carpetas (solo en entorno de desarrollo):

```bash
sudo chmod -R 777 /var/www/miweb1
sudo chmod -R 777 /var/www/miweb2
```

## 4. Crea los ficheros de VirtualHost

Nos situamos en el directorio de configuración de sitios:

```bash
cd /etc/apache2/sites-available
```

### VirtualHost de miweb1 (con navegación de directorio activada)

```bash
sudo cp 000-default.conf miweb1.conf
sudo nano miweb1.conf
```

Edita el contenido para que quede así (sustituye `<IP>` por tu IP real):

```html
<VirtualHost *:80>
    ServerName miweb1.<IP>.nip.io
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/miweb1

    ErrorLog ${APACHE_LOG_DIR}/miweb1-error.log
    CustomLog ${APACHE_LOG_DIR}/miweb1-access.log combined
</VirtualHost>

<Directory /var/www/miweb1>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

!!! note Nota
    La opción **`Indexes`** dentro de `Options` hace que, si Apache no encuentra el archivo de índice configurado, **muestre un listado navegable** con todos los archivos del directorio. En este VirtualHost la dejamos activada a propósito.

### VirtualHost de miweb2 (sin navegación de directorio y con índice personalizado)

```bash
sudo cp 000-default.conf miweb2.conf
sudo nano miweb2.conf
```

Edita el contenido para que quede así:

```html
<VirtualHost *:80>
    ServerName miweb2.<IP>.nip.io
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/miweb2

    ErrorLog ${APACHE_LOG_DIR}/miweb2-error.log
    CustomLog ${APACHE_LOG_DIR}/miweb2-access.log combined
</VirtualHost>

<Directory /var/www/miweb2>
    Options FollowSymLinks
    DirectoryIndex miweb.html
    AllowOverride None
    Require all granted
</Directory>
```

!!! note Nota
    - Al **no incluir `Indexes`** dentro de `Options`, si Apache no encuentra el archivo de índice, devolverá un error `403 Forbidden` en lugar de mostrar el listado de archivos.
    - La directiva **`DirectoryIndex miweb.html`** le indica a Apache que, para este directorio en concreto, el archivo de índice no es `index.html` sino `miweb.html`.

## 5. Activa los VirtualHosts

```bash
sudo a2ensite miweb1.conf
sudo a2ensite miweb2.conf
```

!!! tip
    Puedes comprobar que se han activado correctamente comprobando que existen los enlaces simbólicos dentro de `/etc/apache2/sites-enabled/`.

(Opcional pero recomendado) Desactiva el sitio por defecto para evitar confusiones:

```bash
sudo a2dissite 000-default.conf
```

Comprueba que la configuración no tiene errores de sintaxis:

```bash
sudo apache2ctl configtest
```

Debería aparecer `Syntax OK`.

Recarga la configuración de Apache para aplicar los cambios:

```bash
sudo systemctl reload apache2
```

## 6. Comprueba el resultado

Desde tu navegador (o usando `wget -qO-` como en la práctica anterior), accede a:

```text
http://miweb1.<IP>.nip.io
http://miweb2.<IP>.nip.io
```

Deberías comprobar que:

1. **`miweb1`** muestra correctamente el `index.html` con el mensaje "Bienvenido a Mi Web 1".
2. **`miweb2`** muestra correctamente `miweb.html` con el mensaje "Bienvenido a Mi Web 2", a pesar de no llamarse `index.html`.

## 7. Comprueba el comportamiento de `Indexes`

Para comprobar la diferencia entre ambos `Directory`, renombra temporalmente los archivos de índice de cada sitio:

```bash
sudo mv /var/www/miweb1/index.html /var/www/miweb1/otro.html
sudo mv /var/www/miweb2/miweb.html /var/www/miweb2/otro.html
```

Accede de nuevo a ambas URLs y responde:

1. Al acceder a `http://miweb1.<IP>.nip.io`, ¿qué ves? ¿Por qué ocurre esto?
2. Al acceder a `http://miweb2.<IP>.nip.io`, ¿qué ves? ¿Por qué ocurre algo distinto?

Deshaz el cambio para dejar los sitios funcionando correctamente:

```bash
sudo mv /var/www/miweb1/otro.html /var/www/miweb1/index.html
sudo mv /var/www/miweb2/otro.html /var/www/miweb2/miweb.html
```

## 8. Preguntas finales

1. ¿Qué directiva del `VirtualHost` es la que permite a Apache distinguir entre `miweb1` y `miweb2` si ambos están en el mismo servidor y el mismo puerto `80`?
2. ¿Qué ventaja tiene usar `nip.io` en un laboratorio sin permisos de administrador frente a modificar el fichero `hosts`?
3. ¿Qué riesgo de seguridad tiene activar la opción `Indexes` en un entorno de producción?
4. ¿Qué pasaría si creamos un tercer VirtualHost con el mismo `ServerName` que `miweb1`? ¿A cuál crees que respondería Apache?
