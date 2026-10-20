<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD02 - WWW y protocolo HTTP/Práctica 2.md -->
# Práctica 2

En esta práctica analizaremos a bajo nivel el establecimiento de conexiones **TCP** y el *handshake* **TLS**, para comprobar de forma práctica los conceptos teóricos vistos en el apartado de protocolos de internet y HTTP.

## Objetivos

- Observar el *3-way handshake* de TCP con una herramienta de captura de tráfico.
- Observar el *handshake* TLS al establecer una conexión HTTPS.
- Medir y comparar los tiempos de respuesta de una conexión HTTP frente a una HTTPS.

## 1. Captura de tráfico con Wireshark

1. Abre **Wireshark** y selecciona la interfaz de red por la que sale el tráfico (Wi-Fi o Ethernet).
2. En el campo de filtro, escribe lo siguiente para capturar solo el tráfico relacionado con el protocolo TCP (sustituye por la IP del servidor que vayas a usar en el filtro si lo deseas):

```text
tcp
```

3. Comienza la captura.
4. Desde un navegador o terminal, accede a `http://www.example.com` (en texto plano).
5. Detén la captura.

Busca en el listado de paquetes capturados los **tres primeros paquetes** de la conexión y responde:

1. ¿Qué *flags* TCP aparecen en el primer paquete? (Debería ser `SYN`)
2. ¿Qué *flags* aparecen en el segundo paquete? (Debería ser `SYN, ACK`)
3. ¿Qué *flags* aparecen en el tercer paquete? (Debería ser `ACK`)
4. ¿En qué paquete aparece ya la petición HTTP (`GET /...`)?

## 2. Captura del *handshake* TLS

1. Limpia la captura anterior y vuelve a iniciar la captura en Wireshark.
2. Esta vez, aplica el filtro:

```text
tls
```

3. Accede desde el navegador a `https://www.example.com`.
4. Detén la captura.

Busca los siguientes paquetes y anota en qué número de paquete aparece cada uno:

| Mensaje del handshake TLS | Nº de paquete |
|------------------------------|:---:|
| Client Hello                  |     |
| Server Hello                  |     |
| Certificate                   |     |
| Application Data (datos cifrados) |     |

!!! tip
    Puedes hacer doble clic sobre el paquete **Certificate** para expandir sus detalles y ver información real del certificado del servidor (emisor, validez, dominio...).

## 3. Comparar tiempos: HTTP vs HTTPS

Utilizando `curl`, mide el tiempo total de una petición HTTP y una HTTPS:

```bash
curl -o /dev/null -s -w "Tiempo total: %{time_total}s\n" http://www.example.com
curl -o /dev/null -s -w "Tiempo total: %{time_total}s\n" https://www.example.com
```

Repite cada prueba 3 veces y anota los resultados:

| Intento | HTTP (s) | HTTPS (s) |
|---------|:---:|:---:|
| 1       |     |     |
| 2       |     |     |
| 3       |     |     |

Responde:

1. ¿La conexión HTTPS tarda más, menos o aproximadamente lo mismo que la HTTP?
2. ¿Por qué crees que ocurre esto, teniendo en cuenta los pasos adicionales del *handshake* TLS?

## 4. Desglose del tiempo de conexión con `curl`

`curl` permite desglosar cuánto tiempo se dedica a cada fase de la conexión:

```bash
curl -o /dev/null -s -w "DNS: %{time_namelookup}s\nConexión TCP: %{time_connect}s\nHandshake TLS: %{time_appconnect}s\nTotal: %{time_total}s\n" https://www.example.com
```

Responde:

1. ¿Cuánto tiempo se dedica a la resolución DNS?
2. ¿Cuánto tiempo se dedica a establecer la conexión TCP (3-way handshake)?
3. ¿Cuánto tiempo adicional añade el *handshake* TLS (`time_appconnect` menos `time_connect`)?

## 5. Simulando la ausencia de HTTPS

Accede a una página que utilice **HTTP Strict Transport Security (HSTS)**, por ejemplo `http://www.google.com`, e intenta forzar la versión HTTP sin redirección automática (puedes usar `curl -v` y fijarte en si hay alguna redirección `301`/`308` hacia HTTPS).

Responde:

1. ¿El servidor te permite navegar por HTTP o te redirige automáticamente a HTTPS?
2. ¿Por qué crees que los sitios web modernos fuerzan el uso de HTTPS incluso si el usuario escribe `http://` en la barra de direcciones?

## Preguntas finales

1. ¿Qué paquetes identificarías como pertenecientes al *3-way handshake* de TCP en una captura de Wireshark?
2. ¿Qué diferencia de tiempo observas entre una conexión HTTP y una HTTPS, y a qué se debe esa diferencia?
3. ¿En qué momento exacto del *handshake* TLS deja de viajar la información en texto plano?
