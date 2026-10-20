<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 1.md -->
# Práctica 1

En esta práctica realizaremos una configuración avanzada sobre el servidor Apache instalado en prácticas anteriores: activaremos módulos adicionales, configuraremos compresión de contenido y habilitaremos HTTPS con un certificado autofirmado.

## Objetivos

- Activar y comprobar módulos de Apache.
- Configurar compresión `mod_deflate`.
- Generar un certificado autofirmado y habilitar HTTPS.

## 1. Comprobar los módulos activos

```bash
apache2ctl -M
```

## 2. Activar el módulo de compresión

```bash
sudo a2enmod deflate
sudo systemctl reload apache2
```

Añade en tu VirtualHost:

```text
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css application/javascript
</IfModule>
```

## 3. Generar un certificado autofirmado

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt
```

## 4. Habilitar el módulo SSL y configurar HTTPS

```bash
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl reload apache2
```

Edita `/etc/apache2/sites-available/default-ssl.conf` para apuntar a tu certificado y tu `DocumentRoot`.

## 5. Comprobación

```bash
curl -vk https://localhost
```

## Preguntas finales

1. ¿Qué diferencia observas en el tamaño de las respuestas con y sin compresión activada?
2. ¿Por qué el navegador muestra una advertencia de seguridad al usar un certificado autofirmado?
