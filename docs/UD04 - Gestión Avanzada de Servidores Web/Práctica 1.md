<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 1.md -->
# Práctica 1 — Autenticación y control de acceso

En esta práctica protegerás un directorio del servidor Apache mediante autenticación básica (usuario/contraseña) y restricción por dirección IP.

## Objetivos

- Crear un fichero de usuarios con `htpasswd`.
- Proteger un directorio con autenticación Basic.
- Combinar autenticación con restricción de acceso por IP.
- Comprobar el resultado con `curl`.

## 1. Preparar el directorio a proteger

```bash
sudo mkdir -p /var/www/html/privado
echo "<h1>Zona privada</h1>" | sudo tee /var/www/html/privado/index.html
```

## 2. Crear el fichero de usuarios

```bash
sudo apt install apache2-utils -y
sudo htpasswd -c /etc/apache2/.htpasswd usuario1
sudo htpasswd /etc/apache2/.htpasswd usuario2
```

## 3. Configurar la protección en el VirtualHost

Edita tu VirtualHost (por ejemplo `000-default.conf`) y añade:

```text
<Directory /var/www/html/privado>
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```

Comprueba la sintaxis y recarga Apache:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

## 4. Comprobar el acceso sin credenciales

```bash
curl -I http://localhost/privado/
```

Deberías obtener un código **401 Unauthorized**.

## 5. Comprobar el acceso con credenciales correctas

```bash
curl -I -u usuario1:tu_contraseña http://localhost/privado/
```

Deberías obtener un código **200 OK**.

## 6. Añadir restricción adicional por IP

Modifica el bloque `<Directory>` para exigir, además del usuario, que la petición provenga de `127.0.0.1`:

```text
<Directory /var/www/html/privado>
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile /etc/apache2/.htpasswd
    <RequireAll>
        Require valid-user
        Require ip 127.0.0.1
    </RequireAll>
</Directory>
```

Recarga Apache y repite las pruebas del apartado 4 y 5.

## 7. Eliminar un usuario del fichero de contraseñas

```bash
sudo htpasswd -D /etc/apache2/.htpasswd usuario2
```

## Preguntas finales

1. ¿Qué código de estado HTTP devuelve Apache cuando las credenciales son incorrectas?
2. ¿Por qué, aunque la autenticación esté correctamente configurada, seguiría siendo inseguro usar esta práctica en producción sin HTTPS?
3. ¿Qué diferencia de comportamiento observas al combinar `Require valid-user` y `Require ip` dentro de `<RequireAll>` frente a usarlos sueltos (sin `RequireAll`)?

