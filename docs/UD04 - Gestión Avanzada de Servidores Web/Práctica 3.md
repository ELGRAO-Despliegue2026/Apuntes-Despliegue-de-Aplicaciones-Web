<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 3.md -->
# Práctica 3 — Despliegue de Apache en un contenedor Docker

En esta práctica desplegarás un servidor Apache dentro de un contenedor Docker, usarás Docker Compose para añadir una base de datos y comprobarás el resultado.

## Objetivos

- Crear un Dockerfile para una imagen de Apache con contenido propio.
- Construir y ejecutar un contenedor a partir de esa imagen.
- Usar Docker Compose para levantar Apache junto a una base de datos.
- Consultar logs y estado del contenedor.

## 1. Instalar Docker (si no está instalado)

```bash
sudo apt update
sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable --now docker
```

## 2. Crear el contenido web

```bash
mkdir -p ~/practica-docker/public-html
echo "<h1>Hola desde un contenedor Docker</h1>" > ~/practica-docker/public-html/index.html
cd ~/practica-docker
```

## 3. Crear el Dockerfile

```dockerfile
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
EXPOSE 80
```

!!!  DANGER Importante: 
    La estructura de carpetas en httpd es diferente que con el apache que descargamos con apt. Aquí la filosofía es tener todo dentro de una única carpeta /usr/local/apache2. El mapeo es el siguiente:

  | Concepto | `httpd` (Docker) | `apache2` (Debian/Ubuntu) |
  |---|---|---|
  | Config principal | `/usr/local/apache2/conf/httpd.conf` | `/etc/apache2/apache2.conf` |
  | Web (DocumentRoot) | `/usr/local/apache2/htdocs` | `/var/www/html` |
  | Logs | `/usr/local/apache2/logs` | `/var/log/apache2` |
  | Módulos | `/usr/local/apache2/modules` | `/usr/lib/apache2/modules` |
  | Ejecutable | `/usr/local/apache2/bin/httpd` | `/usr/sbin/apache2` |
  | Nombre del binario | `httpd` | `apache2` |
  | Activar un módulo | Descomentar `LoadModule` en `httpd.conf` | `a2enmod nombre` |
  | Activar un sitio | Editar `httpd.conf` o incluir un archivo | `a2ensite nombre` |
  | Config modular | Un archivo grande + `extra/` | Muchos archivos con enlaces simbólicos |

!!! TIP
    Diferencias importantes en la práctica
    No existen a2enmod, a2ensite ni sites-available en la imagen httpd. Esas herramientas son específicas de Debian. En httpd todo se hace editando httpd.conf o añadiendo líneas Include.
    Logs: la imagen httpd redirige los logs a stdout/stderr (enlaces a /proc/self/fd/1 y /proc/self/fd/2), de modo que docker logs o el servicio de logs de la nube los recoge directamente. En una instalación con apt van a archivos.
    Usuario: en httpd el proceso corre como daemon, en Debian como www-data. Importa si ajustas permisos de archivos.
    VirtualHosts: en httpd los pones en conf/extra/httpd-vhosts.conf (descomentando su Include) o directamente en httpd.conf. En Debian, un archivo por sitio en sites-available/.
  

## 4. Construir y ejecutar la imagen

```bash
docker build -t practica-apache .
docker run -d -p 8080:80 --name practica-apache practica-apache
```

Comprueba que responde correctamente:

```bash
curl http://localhost:8080
```

## 5. Consultar logs y estado del contenedor

```bash
docker ps
docker logs practica-apache
docker exec -it practica-apache bash
```

## 6. Añadir una base de datos con Docker Compose

Crea un fichero `compose.yaml`:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: ejemplo
      MYSQL_DATABASE: practica
```

Levanta los servicios:

```bash
docker compose up -d
docker compose ps
```

## 7. Detener y limpiar

```bash
docker compose down
docker rm -f practica-apache
docker rmi practica-apache
```

## Preguntas finales

1. ¿Qué diferencia hay entre `docker run` y `docker compose up` a la hora de gestionar varios servicios relacionados?
2. ¿Por qué el contenedor `web` no necesita tener instalado MySQL para poder comunicarse con el contenedor `db`?
3. ¿Qué ventaja ofrece este despliegue frente a instalar Apache y MySQL directamente en el sistema operativo anfitrión?
4. ¿Qué comando usarías para ver los logs de error de Apache generados dentro del contenedor sin necesidad de acceder a una shell?
