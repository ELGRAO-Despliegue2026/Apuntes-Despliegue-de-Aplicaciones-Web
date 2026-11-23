
# Reto 1 Hosting virtual y HTTPS con certificados

Configura dos dominios (usa nip.io) en el mismo Apache mediante virtual hosts, y nginx como único punto de entrada que redirija según el server_name. Genera certificados autofirmados con OpenSSL, activa HTTPS en nginx (terminación SSL), redirige todo el tráfico HTTP a HTTPS y haz que la comunicación interna con Apache sea HTTP. Añade autenticación básica (htpasswd) a una de las rutas.


# Reto 2: Proxy inverso con nginx delante de Apache

Despliega una página estática en Apache y configura nginx como proxy inverso en el puerto 8090, de forma que los usuarios solo accedan por nginx. Comprueba que Apache no responde a peticiones externas directas (restringe Listen a localhost o a la red interna). Añade las cabeceras X-Real-IP y X-Forwarded-For y verifica en los logs de Apache que se registra la IP real del cliente (módulo mod_remoteip).

Preguntas: 

- Para qué sirve el modulo remoteip
- Qué hace RemoteIPHeader X-Forwarded-For?
- Qué hace RemoteIPTrustedProxy?
- Qué archivos hay en la carpeta conf.d de nginx?
- Qué hace proxy_set_header?

# Reto 3: Balanceo de carga con pesos y comprobdockeración de fallos

Despliega la misma aplicación en tres servidores Apache, cada uno mostrando su número (contenedor 1 muestra Servidor 1 etc.). Configura nginx como balanceador con un upstream donde un servidor tenga más peso (weight=3), otro peso normal y el tercero como backup. Apaga servidores uno a uno y demuestra que el servicio sigue disponible y cómo se redistribuye el tráfico. Prueba también ip_hash para sesiones persistentes y explica la diferencia.


Pasos: 
- Crea una red de contenedores utilizando docker network create <NOMBRE_DE_LA_RED> y ejecuta los contenedores en esa red utilizando el flag --network <NOMBRE_DE_LA_RED> en el comando docker run ejecutando un contenedor docker. 

- Para ejecutar código dentro de docker usa `docker exec <NOMBRE_CONTENEDOR> <INSTRUCCIÓN>`. Por ejemplo, para poder instalar curl dentro de los contenedores directamente puedes hacer:

`docker exec <nombre_contenedor> apt update`
`docker exec <nombre_contenedor> apt install curl -y`

- Para el contenedor nginx utiliza la imagen nginx:alpine
- Crea el contenedor nginx en la misma red que los contenedores apache. 
- Asignar los pesos 

# Reto 4: Servir contenido estático con nginx y dinámico con Apache (PHP)

Despliega una aplicación PHP (por ejemplo, un CRUD sencillo o WordPress) en Apache con mod_php o PHP-FPM. Configura nginx para servir directamente los ficheros estáticos (/css, /js, /img) con caché y compresión gzip, y que solo reenvíe a Apache las peticiones dinámicas (.php). Demuestra con curl -I y los logs qué servidor atiende cada petición.


# Reto 5: Arquitectura completa con alta disponibilidad y base de datos

Monta un entorno de tres capas: nginx como balanceador, dos servidores Apache con una aplicación (por ejemplo, un gestor de tareas en PHP o Node) y un servidor de base de datos MySQL/MariaDB compartido. Resuelve el problema de las sesiones entre servidores (sesiones en base de datos o Redis), configura health checks y un fail_timeout en nginx, y simula la caída de un nodo mientras haces peticiones continuas con ab o wrk. Documenta el resultado con un diagrama y una tabla de tiempos de respuesta antes y después del fallo.