<!-- filepath: /home/lenovo-carles/GithubDAW/Apuntes-Despliegue-de-Aplicaciones-Web/docs/UD06 - Servidores de transferencia de archivos/Práctica 1.md -->
# Práctica 1

En esta práctica instalarás y configurarás un servidor SFTP y probarás distintas herramientas de transferencia de archivos (SFTP, SCP y Rsync) entre tu máquina y una máquina virtual.

## Objetivos

- Configurar acceso SFTP a un servidor.
- Transferir ficheros usando SCP.
- Sincronizar una carpeta completa con Rsync.

## 1. Comprobar el servicio SSH

```bash
sudo systemctl status ssh
```

## 2. Conexión SFTP

```bash
sftp usuario@<IP_VM>
put fichero_prueba.txt
ls
exit
```

## 3. Copia con SCP

```bash
scp fichero_prueba.txt usuario@<IP_VM>:/home/usuario/
scp usuario@<IP_VM>:/home/usuario/fichero_prueba.txt ./copia.txt
```

## 4. Sincronización con Rsync

```bash
mkdir carpeta_local
echo "Hola Rsync" > carpeta_local/nota.txt

rsync -avz carpeta_local/ usuario@<IP_VM>:/home/usuario/carpeta_remota/
```

Modifica el fichero local y vuelve a sincronizar, comprobando que solo se transfieren los cambios.

## Preguntas finales

1. ¿Qué diferencias de seguridad existen entre FTP y SFTP?
2. ¿Por qué Rsync es más eficiente que SCP cuando se sincroniza repetidamente la misma carpeta?
