<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD04 - Gestión Avanzada de Servidores Web/Práctica 4.md -->
# Práctica 4 — Gestión y monitorización de logs

En esta práctica configurarás un formato de log personalizado en Apache, la rotación de logs y habilitarás `mod_status` para monitorizar el servidor en tiempo real.

## Objetivos

- Configurar un formato de log personalizado (`LogFormat`).
- Configurar la rotación de logs con `logrotate`.
- Activar y consultar `mod_status`.
- Analizar el log de acceso para detectar patrones.

## 1. Configurar un formato de log personalizado

Edita el VirtualHost (o `apache2.conf`) y añade:

```text
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" practica
CustomLog ${APACHE_LOG_DIR}/practica-access.log practica
```

Comprueba y recarga:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

## 2. Generar tráfico de prueba

```bash
for i in {1..20}; do curl -s http://localhost > /dev/null; done
curl -s http://localhost/no-existe > /dev/null
```

## 3. Consultar el log generado

```bash
sudo tail -n 30 /var/log/apache2/practica-access.log
```

Identifica en el log:

- Una petición correcta (código 200).
- La petición al recurso inexistente (código 404).

## 4. Configurar la rotación de logs

Crea el fichero `/etc/logrotate.d/practica-apache`:

```text
/var/log/apache2/practica-access.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

Simula la rotación manualmente:

```bash
sudo logrotate -f /etc/logrotate.d/practica-apache
ls -la /var/log/apache2/ | grep practica
```

## 5. Activar `mod_status`

```bash
sudo a2enmod status
```

Añade en la configuración:

```text
<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
</Location>
```

```bash
sudo systemctl reload apache2
curl http://localhost/server-status
```

## 6. (Opcional) Análisis básico con comandos de shell

Contar cuántas peticiones ha recibido cada código de estado:

```bash
awk '{print $9}' /var/log/apache2/practica-access.log | sort | uniq -c | sort -nr
```

Listar las 5 IPs con más peticiones:

```bash
awk '{print $1}' /var/log/apache2/practica-access.log | sort | uniq -c | sort -nr | head -5
```

## Preguntas finales

1. ¿Qué información adicional aporta el formato `combined`/`practica` frente al formato de log por defecto de Apache?
2. ¿Por qué es importante configurar la rotación de logs en un servidor en producción?
3. ¿Qué riesgo de seguridad existiría si se dejara accesible `/server-status` sin restricción por IP?
4. ¿Qué ventaja tendría enviar estos logs a una herramienta centralizada como el Elastic Stack frente a revisarlos manualmente con `tail`?
