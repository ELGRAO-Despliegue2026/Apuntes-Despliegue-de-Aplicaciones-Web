# Práctica 1: Creación y Configuración de Máquina Virtual con Debian 13 Mínimo
---

## Índice
1. [Requisitos previos](#1-requisitos-previos)
2. [Paso 1: Comprobación de la Arquitectura del Sistema Anfitrión](#paso-1-comprobación-de-la-arquitectura-del-sistema-anfitrión)
3. [Paso 2: Descarga de la Imagen ISO de Debian 13](#paso-2-descarga-de-la-imagen-iso-de-debian-13)
4. [Paso 3: Creación de la Máquina Virtual en VirtualBox](#paso-3-creación-de-la-máquina-virtual-en-virtualbox)
5. [Paso 4: Instalación Paso a Paso de Debian 13 Mínimo](#paso-4-instalación-paso-a-paso-de-debian-13-mínimo)
6. [Paso 5: Verificación del Entorno sin GUI](#paso-5-verificación-del-entorno-sin-gui)


---

## Paso 1: Comprobación de la Arquitectura del Sistema Anfitrión

Antes de descargar la ISO de Debian 13, es fundamental determinar qué arquitectura tiene el procesador de tu equipo para elegir la imagen adecuada.

1. Abre la terminal o consola de comandos en tu equipo anfitrión.
2. Ejecuta el comando:

```bash
uname -m
```

3. **Interpretación del resultado:**
   * **`x86_64`**: Corresponde a la arquitectura **`amd64`** (Intel o AMD de 64 bits). *Es el resultado más común en PCs y portátiles habituales.*
   * **`aarch64`**: Corresponde a la arquitectura **`arm64`** (Apple Silicon M1/M2/M3/M4, PCs con Qualcomm Snapdragon, Raspberry Pi, etc.).

---

## Paso 2: Descarga de la Imagen ISO de Debian 13

1. Accede a la página oficial de descargas de **Debian**.
2. Dirígete a la sección de la versión de pruebas/testing (**Debian 13 "Trixie"**) o descarga la ISO de tipo **Netinst** (instalación por red).
3. Selecciona la arquitectura obtenida en el **Paso 1**:
   * Si el resultado fue `x86_64` $\rightarrow$ Descargar ISO versión **`amd64`**.
   * Si el resultado fue `aarch64` $\rightarrow$ Descargar ISO versión **`arm64`**.

---

## Paso 3: Creación de la Máquina Virtual en VirtualBox

1. Abre **VirtualBox** y haz clic en el botón **Nueva**.
2. **Nombre y sistema operativo:**
   * **Nombre:** `Debian13-Servidor-DAW`
   * **Imagen ISO:** Selecciona el archivo ISO de Debian 13 descargado.
   * **Tipo:** Linux
   * **Versión:** Debian (64-bit) o Debian (ARM64) según tu procesador.
3. **Hardware:**
   * **Memoria RAM:** Asigna mínimo `1024 MB` (1 GB) o `2048 MB` (2 GB). Al no llevar entorno gráfico, consumirá muy pocos recursos.
   * **Procesadores:** 1 o 2 vCPUs.
4. **Disco Duro Virtual:**
   * Selecciona **Crear un disco duro virtual ahora**.
   * **Tamaño:** `15,00 GB` (suficiente para el sistema base y el servidor web).
5. **Configuración de Red (Importante):**
   * Selecciona la máquina virtual creada y entra en **Configuración > Red**.
   * En el **Adaptador 1**, asegúrate de tener la opción "NAT".
   En el Reenvio de puertos tendrás que añadir dos reglas:

| **Nombre** | **Protocolo** | **IP anfitrión** | **Puerto anfitrión** | **IP invitado** | **Puerto invitado** |
| --- | --- | --- | --- | --- | --- |
| HTTP | TCP | 127.0.0.1 | 8080 | 10.0.2.15 | 80 |
| SSH | TCP | | 2222 |  | 22 |


---

## Paso 4: Instalación Paso a Paso de Debian 13 Mínimo

1. Inicia la máquina virtual con la ISO insertada.
2. En el menú de arranque de GRUB, selecciona **Install** o **Graphical Install**.
3. **Configuración regional:**
   * Elige el idioma (ej. *Spanish - Español*).
   * Selecciona tu ubicación y la distribución del teclado (*Español*).
4. **Configuración de Red y Usuarios:**
   * **Nombre del equipo (Hostname):** `servidor-apache`
   * **Nombre de dominio:** *(Dejar en blanco)*
   * **Contraseña de root:** Introduce una contraseña segura para el superusuario y confírmala.
   * **Usuario no privilegiado:** Crea una cuenta de usuario estándar (ej. `alumno`) y asigna su clave.
5. **Particionado del disco:**
   * Método de particionado: **Guiado - usar todo el disco**.
   * Selecciona el disco duro virtual creado.
   * Esquema: **Todos los ficheros en una sola partición (recomendado para novatos)**.
   * Selecciona **Finalizar el particionado y escribir los cambios en el disco** y confirma marcando **Sí**.
6. **Selección de Programas (*Paso Clave para versión Mínima*):**
   * Cuando aparezca la pantalla **"Selección de programas" / "Tasksel"**:
     * **Desmarca:** `Entorno de escritorio Debian` / `Debian desktop environment`.
     * **Desmarca:** Cualquier GUI adicional (`GNOME`, `XFCE`, `KDE`, etc.).
     * **Marca:** `Utilidades estándar del sistema` (*standard system utilities*).
     * **Marca (Opcional):** `SSH server` (útil para conectarse remotamente).

> **Navegación:** Usa las flechas para moverte, la **Barra Espaciadora** para marcar/desmarcar casillas, y la tecla **Tabulador** para seleccionar el botón *Continuar*.

7. **Instalación del cargador de arranque GRUB:**
   * Selecciona **Sí** para instalar GRUB en el unidad principal.
   * Elige el dispositivo `/dev/sda` o la unidad virtual correspondiente.
8. **Finalización:**
   * Cuando indique que la instalación ha completado, extrae la ISO virtual y haz clic en **Continuar** para reiniciar.

---

## Paso 5: Verificación del Entorno sin GUI

Una vez reiniciada la máquina virtual, debes comprobar que el sistema ha arrancado correctamente en modo texto/consola:

1. Verás en pantalla la solicitud de login por CLI:
   ```text
   servidor-apache login:
   ```
2. Inicia sesión con el usuario `root` o el usuario creado.
3. Comprueba la versión instalada ejecutando:
   ```bash
   cat /etc/debian_version
   ```
4. Verifica la interfaz de red e IP asignada ejecutando:
   ```bash
   ip a
   ```