<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD07 - Servicios red implicados en el despliegue de una aplicación Web/Práctica 1.md -->
# Práctica 1

En esta práctica configurarás y comprobarás el funcionamiento de varios servicios de red necesarios en el despliegue de una aplicación web: DNS, NTP y correo (SMTP/IMAP).

## Objetivos

- Consultar registros DNS de un dominio real.
- Comprobar y forzar la sincronización horaria con NTP.
- Enviar un correo de prueba mediante un servidor SMTP local.

## 1. Consultas DNS

```bash
dig www.ejemplo.com
dig MX ejemplo.com
```

Responde:

1. ¿Qué dirección IP se ha devuelto para el registro `A`?
2. ¿Qué servidor de correo aparece en el registro `MX`?

## 2. Comprobar la sincronización NTP

```bash
chronyc tracking
chronyc sources -v
```

Responde: ¿con qué servidor NTP está sincronizado tu equipo y cuál es el desfase actual (*offset*)?

## 3. Instalar Postfix y Dovecot

```bash
sudo apt install postfix dovecot-imapd -y
```

## 4. Enviar un correo de prueba

```bash
echo "Mensaje de prueba" | mail -s "Prueba SMTP" usuario@localhost
```

## 5. Comprobar la recepción vía IMAP

```bash
sudo systemctl status dovecot
```

## Preguntas finales

1. ¿Qué ocurriría si la hora del servidor no estuviera sincronizada correctamente al generar un certificado TLS?
2. ¿Qué diferencia fundamental existe entre consultar un registro `A` y un registro `MX`?
