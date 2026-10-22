<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD03 - Introducción a servidores Web. Apache/Práctica 4.md -->
# Práctica 4 — mod_rewrite

En esta práctica aprenderás a activar y utilizar el módulo **mod_rewrite** de Apache, que permite reescribir y redirigir URLs de forma flexible mediante expresiones regulares.

## ¿Qué es mod_rewrite?

**mod_rewrite** es un módulo de Apache que permite modificar las URLs solicitadas por el cliente **antes** de que el servidor las procese. Se utiliza habitualmente para:

- Crear **URLs amigables** (ej. `/producto/15` en lugar de `/producto.php?id=15`).
- Redirigir peticiones de un dominio/ruta a otro.
- Forzar el uso de HTTPS o de `www`.
- Bloquear el acceso según ciertas condiciones (user-agent, referer...).

## Objetivos

- Activar el módulo `mod_rewrite`.
- Permitir el uso de reglas de reescritura mediante `.htaccess` o en el propio VirtualHost.
- Crear reglas de redirección y de reescritura de URLs.

## 1. Activar el módulo

```bash
sudo a2enmod rewrite
sudo systemctl reload apache2
```

Comprueba que el módulo está activo:

```bash
apache2ctl -M | grep rewrite
```

Debería aparecer `rewrite_module (shared)`.

## 2. Permitir el uso de reglas en el directorio

Para que las reglas de reescritura funcionen dentro de un `.htaccess`, el `<Directory>` correspondiente debe permitir sobrescribir la configuración:

```text
<Directory /var/www/html>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

!!! note
    `AllowOverride All` permite que un fichero `.htaccess` en el directorio pueda definir directivas como `RewriteEngine` y `RewriteRule`. Sin esta directiva, Apache ignorará el `.htaccess`.

Recarga Apache tras el cambio:

```bash
sudo systemctl reload apache2
```

## 3. Crear el fichero `.htaccess`

Dentro de `/var/www/html`, crea el fichero `.htaccess`:

```bash
sudo nano /var/www/html/.htaccess
```

Y activa el motor de reescritura:

```text
RewriteEngine On
```

!!! tip
    También podrías escribir todas las reglas directamente dentro del `VirtualHost` en lugar de en un `.htaccess`, lo cual es más eficiente ya que Apache no tiene que comprobar su existencia en cada petición. Para ello, usa un bloque `<Directory>` con `RewriteEngine On` y las reglas correspondientes.

## 4. Redirección simple

Vamos a redirigir la ruta `/viejo` hacia `/nuevo`.

Crea el contenido necesario:

```bash
echo "<h1>Página nueva</h1>" | sudo tee /var/www/html/nuevo.html
```

Añade la regla en el `.htaccess`:

```text
RewriteEngine On
RewriteRule ^viejo$ /nuevo.html [R=301,L]
```

- `^viejo$`: expresión regular que coincide exactamente con la ruta `/viejo`.
- `[R=301,L]`: `R=301` indica una redirección permanente, `L` (*Last*) indica que, si la regla coincide, no se evalúen más reglas.

Comprueba el resultado:

```bash
curl -I http://localhost/viejo
```

Deberías obtener un código **301** con la cabecera `Location: /nuevo.html`.

## 5. URLs amigables con parámetros

Simularemos una URL amigable que internamente reescribe a un script PHP con parámetros (aunque no tengas PHP instalado, puedes comprobar la reescritura con un fichero de prueba).

```bash
echo "<h1>Mostrando producto</h1>" | sudo tee /var/www/html/producto.html
```

Añade la regla:

```text
RewriteEngine On
RewriteRule ^producto/([0-9]+)$ /producto.html?id=$1 [L]
```

- `([0-9]+)`: captura uno o más dígitos de la URL.
- `$1`: representa el primer grupo capturado, que se añade como parámetro `id`.

Comprueba:

```bash
curl -I http://localhost/producto/15
```

Debería devolver **200 OK**, sirviendo realmente `producto.html` aunque la URL solicitada sea `/producto/15`.

## 6. Forzar HTTPS con mod_rewrite

Otra utilidad muy habitual de `mod_rewrite` es forzar que todo el tráfico HTTP se redirija a HTTPS (alternativa a usar `Redirect permanent` visto en la UD04):

```text
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

- `RewriteCond %{HTTPS} off`: la regla siguiente solo se aplica si la conexión **no** es HTTPS.
- `%{HTTP_HOST}` y `%{REQUEST_URI}`: variables que contienen el dominio solicitado y la ruta completa de la petición.

## 7. Bloquear el acceso según el User-Agent

```text
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} ^BadBot [NC]
RewriteRule ^.*$ - [F,L]
```

- `[NC]`: la comparación no distingue mayúsculas/minúsculas (*No Case*).
- `[F,L]`: `F` devuelve un código **403 Forbidden**.

Comprueba el bloqueo:

```bash
curl -I -A "BadBot" http://localhost/
```

## 8. Ver el registro de reescrituras (depuración)

Para depurar reglas complejas, puedes activar el log de reescritura mediante el módulo de log:

```text
LogLevel alert rewrite:trace3
```

Añádelo al `VirtualHost` y consulta después:

```bash
sudo tail -f /var/log/apache2/error.log
```

## Preguntas finales

1. ¿Qué ocurre si olvidas añadir la bandera `[L]` en una regla de reescritura y hay más reglas después?
2. ¿Por qué es necesario `AllowOverride All` para que funcionen las reglas definidas en un `.htaccess`?
3. ¿Qué diferencia hay entre `RewriteRule` y `RewriteCond`?
4. ¿Qué ventaja de rendimiento tendría mover las reglas del `.htaccess` directamente al `VirtualHost`?
5. En la regla de URLs amigables del apartado 5, ¿qué representa `$1` y de dónde proviene su valor?
