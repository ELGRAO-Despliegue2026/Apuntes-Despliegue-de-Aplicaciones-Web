<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 3.md -->
# Práctica 1— Contenedores y pruebas de rendimiento

En esta práctica contenerizarás una aplicación Tomcat con Docker y realizarás una prueba de carga básica sobre ella.

## Objetivos

- Crear una imagen Docker propia de Tomcat con una aplicación desplegada.
- Levantar el contenedor y comprobar su funcionamiento.
- Realizar una prueba de carga con Apache Bench y analizar los resultados.

## 1. Preparar la aplicación

Reutiliza el WAR `miapp.war` creado en la Práctica 1 (o genera uno nuevo siguiendo los mismos pasos).

## 2. Crear el `Dockerfile`

```bash
mkdir docker-tomcat && cd docker-tomcat
cp ../miapp.war .
```

```dockerfile
FROM tomcat:10-jdk17

RUN rm -rf /usr/local/tomcat/webapps/examples \
           /usr/local/tomcat/webapps/docs \
           /usr/local/tomcat/webapps/host-manager

COPY miapp.war /usr/local/tomcat/webapps/miapp.war

EXPOSE 8080
```

## 3. Construir y ejecutar la imagen

```bash
docker build -t miapp-tomcat:1.0 .
docker run -d -p 8080:8080 --name miapp miapp-tomcat:1.0
```

## 4. Comprobar el funcionamiento

```bash
docker ps
docker logs -f miapp
curl -I http://localhost:8080/miapp/
```

## 5. Prueba de carga con Apache Bench

```bash
sudo apt install apache2-utils -y
ab -n 500 -c 50 http://localhost:8080/miapp/
```

Anota los siguientes datos del resultado:

| Métrica                     | Valor obtenido |
|-------------------------------|------------------|
| Requests per second           |                  |
| Time per request (media)      |                  |
| Failed requests               |                  |

## 6. Monitorizar el consumo de recursos del contenedor durante la prueba

En otra terminal, mientras se ejecuta `ab`:

```bash
docker stats miapp
```

Anota el consumo aproximado de **CPU** y **memoria** durante la prueba de carga.

## 7. Repetir la prueba aumentando la concurrencia

```bash
ab -n 1000 -c 200 http://localhost:8080/miapp/
```

Compara los resultados con la prueba anterior.

## 8. (Opcional) Orquestar con Docker Compose

Crea un `docker-compose.yml` que levante la aplicación junto con una base de datos MySQL:

```yaml
version: "3.8"
services:
  tomcat:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - bd

  bd:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: "contraseña"
      MYSQL_DATABASE: "mibasedatos"
```

```bash
docker compose up -d
docker compose ps
```

## Preguntas finales

1. ¿Qué diferencia de rendimiento observas al aumentar la concurrencia de 50 a 200 peticiones simultáneas?
2. ¿Qué ventaja aporta eliminar las aplicaciones de ejemplo (`examples`, `docs`, `host-manager`) en la imagen Docker?
3. ¿Qué ocurriría si el contenedor se quedase sin memoria durante la prueba de carga? ¿Cómo lo detectarías con `docker stats`?
4. ¿Qué ventaja tiene usar `docker-compose` frente a arrancar manualmente cada contenedor por separado?
