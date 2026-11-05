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
