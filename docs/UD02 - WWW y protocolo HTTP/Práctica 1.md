<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD02 - WWW y protocolo HTTP/Práctica 1.md -->
# Práctica 1

En esta práctica analizaremos peticiones y respuestas HTTP reales utilizando las herramientas de desarrollador del navegador y la herramienta de línea de comandos `curl`.

## Objetivos

- Observar el formato real de una petición y una respuesta HTTP.
- Identificar métodos, códigos de estado y cabeceras.
- Comprobar la diferencia entre una petición HTTP y una HTTPS.

## 1. Inspeccionar peticiones con el navegador

1. Abre tu navegador y pulsa `F12` (o clic derecho → *Inspeccionar*) para abrir las herramientas de desarrollador.
2. Ve a la pestaña **Network/Red**.
3. Marca la casilla de **Preserve log** (Registros Persistentes en firefox) si está disponible.
4. Visita la página `http://www.example.com` (fíjate que es HTTP, sin "s").
5. Haz clic sobre la primera petición de la lista (normalmente el documento HTML principal).

Responde en tu cuaderno de prácticas:

1. ¿Qué **método** HTTP se ha utilizado?
2. ¿Qué **código de estado** ha devuelto el servidor?
3. ¿Qué cabecera indica el tipo de contenido devuelto (`Content-Type`)?
4. ¿Aparece la cabecera `Host`? ¿Qué valor tiene?

## 2. Repite la prueba con HTTPS

Visita ahora `https://www.google.com` y repite el análisis de la petición principal:

1. ¿En qué puerto se ha establecido la conexión (pista: está en la respuesta el alt-svc)?
2. Haz clic en el candado de la barra de direcciones. ¿Qué Autoridad de Certificación ha emitido el certificado?
3. ¿Para qué dominio es válido el certificado?

## 3. Peticiones HTTP con `curl`

Abre una terminal (en Linux/Mac, o Git Bash en Windows) y ejecuta:

```bash
curl -v http://www.example.com
```

La opción `-v` (*verbose*) muestra el detalle completo de la petición y la respuesta, incluyendo:

- Los mensajes de establecimiento de la conexión.
- La petición enviada (líneas que empiezan por `>`).
- La respuesta recibida (líneas que empiezan por `<`).

Responde:

1. ¿Qué cabeceras ha enviado `curl` en la petición?
2. ¿Qué código de estado y qué cabeceras ha devuelto el servidor?

## 4. Comprobando solo las cabeceras

Repite la petición, pero mostrando solo las cabeceras de la respuesta, sin el cuerpo:

```bash
curl -I http://www.example.com
```

## 5. Petición HTTPS con `curl`

```bash
curl -v https://www.example.com
```

Compara la salida con la de la petición HTTP del apartado 3. Responde:

1. ¿Qué nuevos pasos aparecen antes de que se realice la petición HTTP (pista: busca referencias a `TLS`, `SSL` o al certificado)?
2. ¿Qué información del certificado puedes ver en la salida de `curl`?

## 6. Códigos de estado

Prueba a realizar peticiones a URLs que generen distintos códigos de estado y anota el resultado:

```bash
curl -I https://httpstat.us/200
curl -I https://httpstat.us/301
curl -I https://httpstat.us/404
curl -I https://httpstat.us/500
```

Completa la siguiente tabla con lo observado:

| URL                         | Código de estado | Significado |
|------------------------------|:---:|----------------|
| httpstat.us/200               |     |                |
| httpstat.us/301               |     |                |
| httpstat.us/404               |     |                |
| httpstat.us/500               |     |                |

## 7. Petición POST con `curl`

```bash
curl -X POST -d "nombre=tunombre&curso=DAW" https://httpbin.org/post
```

Observa la respuesta JSON devuelta e identifica dónde aparecen los datos que has enviado.

## Preguntas finales

1. ¿Qué diferencias principales has observado entre una petición HTTP y una HTTPS?
2. ¿Por qué crees que es importante comprobar el certificado de un sitio web antes de introducir datos sensibles (contraseñas, tarjetas de crédito...)?
3. ¿Qué ventaja tiene usar `curl` frente a las herramientas de desarrollador del navegador a la hora de automatizar pruebas?
