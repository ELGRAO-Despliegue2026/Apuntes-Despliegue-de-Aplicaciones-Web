<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD03 - Introducción a servidores Web. Apache/Práctica 4.md -->
# Práctica 4 — mod_rewrite

En esta práctica aprenderás a activar y utilizar el módulo **mod_rewrite** de Apache, que permite reescribir y redirigir URLs de forma flexible mediante expresiones regulares.

## Introducción a las expresiones regulares (regex)

`mod_rewrite` basa todas sus reglas en **expresiones regulares (regex)**: patrones de texto que permiten describir y buscar coincidencias dentro de una cadena (en este caso, la URL solicitada).

Una expresión regular se construye combinando **caracteres literales** (que deben coincidir exactamente) con **caracteres especiales** (que tienen un significado propio dentro de la regex). Por ejemplo, en el patrón `^producto/([0-9]+)$`:

- `producto/` son caracteres literales.
- `^`, `$`, `()`, `[0-9]` y `+` son caracteres especiales con un significado concreto.

A continuación se resumen los caracteres especiales más utilizados en las reglas de `mod_rewrite`:

| Carácter | Significado                                                      | Ejemplo                  |
|:--------:|---------------------------------------------------------------------|------------------------------|
| `^`      | Indica el **inicio** de la cadena                                   | `^producto` coincide con `producto/15`, pero no con `mi-producto` |
| `$`      | Indica el **final** de la cadena                                    | `viejo$` coincide con `/muy-viejo`, pero no con `viejo-articulo` |
| `.`      | Coincide con **cualquier carácter** (excepto salto de línea)         | `a.c` coincide con `abc`, `axc`, `a1c`... |
| `*`      | El elemento anterior se repite **cero o más veces**                  | `ab*` coincide con `a`, `ab`, `abb`, `abbb`... |
| `+`      | El elemento anterior se repite **una o más veces**                   | `[0-9]+` coincide con `5`, `42`, `12345`, pero no con una cadena vacía |
| `?`      | El elemento anterior aparece **cero o una vez** (opcional)           | `colou?r` coincide con `color` y `colour` |
| `[...]`  | Define una **clase de caracteres**: cualquiera de los indicados      | `[0-9]` coincide con un solo dígito; `[a-z]` con una letra minúscula |
| `[^...]` | **Negación** de una clase de caracteres                              | `[^0-9]` coincide con cualquier carácter que **no** sea un dígito |
| `(...)`  | Define un **grupo de captura**, reutilizable después como `$1`, `$2`... | `([0-9]+)` captura un número para usarlo en la sustitución |
| `\|`     | Operador **OR**: coincide con una opción u otra                      | `gato\|perro` coincide con `gato` o con `perro` |
| `{n}`    | El elemento anterior se repite **exactamente `n` veces**             | `[0-9]{4}` coincide con exactamente 4 dígitos (ej. un año) |
| `{n,m}`  | El elemento anterior se repite **entre `n` y `m` veces**             | `[0-9]{2,4}` coincide con entre 2 y 4 dígitos |
| `\`      | **Escapa** un carácter especial para tratarlo como literal           | `\.` coincide con un punto literal, no con "cualquier carácter" |

!!! tip "Comprobar expresiones regulares"
    Puedes probar y visualizar tus expresiones regulares antes de usarlas en Apache con herramientas online como [regex101.com](https://regex101.com), que además explican paso a paso qué hace cada parte del patrón.

Con estas bases ya podemos entender las reglas de `mod_rewrite` que usaremos a lo largo de la práctica.

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

Comprueba que el módulo está activo: (Ojo, según desde que usuario instalaste apache2 quizás debes hacer `sudo -i`)

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

`^viejo$`: expresión regular que coincide exactamente con la ruta `/viejo`.

`[R=301,L]`: `R=301` indica una redirección permanente (respuesta HTTP), `L` (*Last*) significa que si la condición se cumple deje de buscar.

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
`([0-9]+)`: captura uno o más dígitos de la URL. Los paréntesis indican el grupo que está siendo "capturado". 

`$1`: representa el primer grupo capturado, que se añade como parámetro `id`.

Si por ejemplo tenemos /(ab)+/([0-9]+) y queremos que se reescriba como num/<número> entonces tendríamos que hacer num/$2 ya que es el segundo grupo capturado (el primero es (ab)+)

Comprueba:

```bash
curl -I http://localhost/producto/15
```

Debería devolver **200 OK**, sirviendo realmente `producto.html` aunque la URL solicitada sea `/producto/15`.


!!! tip "Comprobar la petición interna reescrita"
    Para ver exactamente qué petición le llega al recurso final (por ejemplo, `producto.html?id=15`), puedes activar el log de reescritura (apartado 8) y consultar el fichero de errores:

    ```bash
    sudo tail -f /var/log/apache2/error.log
    ```

    Verás una línea similar a:

    ```text
    rewrite 'producto/15' -> '/producto.html?id=15'
    ```


## 6. Bloquear el acceso según el User-Agent

```text
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} ^BadBot [NC]
RewriteRule ^.*$ - [F,L]
```

`[NC]`: la comparación no distingue mayúsculas/minúsculas (*No Case*).

`[F,L]`: `F` devuelve un código **403 Forbidden**.

Comprueba el bloqueo:

```bash
curl -I -A "BadBot" http://localhost/
```

## 7. Ver el registro de reescrituras (depuración)

Para depurar reglas complejas, puedes activar el log de reescritura mediante el módulo de log:

```text
LogLevel alert rewrite:trace3
LogLevel warn rewrite:trace3
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
