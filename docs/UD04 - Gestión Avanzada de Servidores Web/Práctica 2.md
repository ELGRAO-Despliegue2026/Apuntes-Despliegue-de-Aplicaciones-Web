<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 2.md -->
# Práctica 2 — Certificados y HTTPS

En esta práctica generarás un certificado autofirmado, configurarás HTTPS en Apache, forzarás la redirección desde HTTP y comprobarás el resultado con distintas herramientas.

## Objetivos

- Generar un certificado autofirmado con OpenSSL.
- Activar el módulo SSL y configurar un VirtualHost HTTPS.
- Forzar la redirección de HTTP a HTTPS.
- Activar la cabecera HSTS.
- Comprobar el funcionamiento con `curl` y `openssl`.

## 1. Generar el certificado autofirmado

```bash
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/apache-selfsigned.key \
  -out /etc/apache2/ssl/apache-selfsigned.crt
```

Durante la generación, indica como *Common Name* (`CN`) el nombre de dominio que vayas a usar (por ejemplo, el de tus prácticas con `nip.io`).

## 2. Activar el módulo SSL

```bash
sudo a2enmod ssl
sudo a2enmod headers
```

## 3. Crear el VirtualHost HTTPS

```bash
cd /etc/apache2/sites-available
sudo cp default-ssl.conf miweb-ssl.conf
sudo nano miweb-ssl.conf
```

Configura el fichero con tu certificado y tu `DocumentRoot`:

```text
<IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerName midominio.nip.io
        DocumentRoot /var/www/html

        SSLEngine on
        SSLCertificateFile /etc/apache2/ssl/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/apache2/ssl/apache-selfsigned.key

        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    </VirtualHost>
</IfModule>
```

Activa el sitio y recarga Apache:

```bash
sudo a2ensite miweb-ssl.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

## 4. Forzar la redirección de HTTP a HTTPS

Edita el VirtualHost del puerto 80 correspondiente al mismo dominio:

```text
<VirtualHost *:80>
    ServerName midominio.nip.io
    Redirect permanent / https://midominio.nip.io/
</VirtualHost>
```

Recarga Apache:

```bash
sudo systemctl reload apache2
```

## 5. Comprobar el resultado

### Comprobar que HTTP redirige a HTTPS

```bash
curl -I http://midominio.nip.io
```

Deberías ver un código **301** y la cabecera `Location` apuntando a la versión HTTPS.

### Comprobar la conexión HTTPS

```bash
curl -vk https://midominio.nip.io
```

### Comprobar la cabecera HSTS

```bash
curl -Ik https://midominio.nip.io | grep -i strict-transport-security
```

### Inspeccionar el certificado con OpenSSL

```bash
openssl s_client -connect midominio.nip.io:443 -servername midominio.nip.io </dev/null | openssl x509 -noout -dates -subject -issuer
```

## 6. (Opcional) Analizar la configuración con SSL Labs

Si tu servidor es accesible públicamente, analiza su configuración TLS en [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/) y anota la calificación obtenida.

## Preguntas finales

1. ¿Qué diferencia observas entre acceder con `curl -v` (sin `-k`) y con `curl -vk` a tu servidor con certificado autofirmado?
2. ¿Qué información del certificado ha mostrado el comando `openssl x509 -noout -dates -subject -issuer`?
3. ¿Por qué es necesario activar el módulo `headers` para poder usar la directiva `Header` (HSTS)?
4. ¿Qué ocurriría si accedieras a `https://midominio.nip.io` desde un navegador en lugar de con `curl`? Descríbelo.
