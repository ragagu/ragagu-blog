---
layout: post
title:  Instalación de subsistema de Windows para Linux (WSL) y Ubuntu 22.04 LTS
description: Instalamos WSL y Ubuntu 22.04 LTS en Windows 11. Mostramos también una serie de comandos útiles para la gestión de WSL.
date:   2022-07-06 12:57:17 +0300
image:  '/images/instalando-ubuntu-22-04-lts-wsl.png'
tags:   [Windows 11, Ubuntu 22.04 LTS, WSL]
---

En el siguiente artículo vamos habilitar WSL, una característica integrada en Windows 11 que nos permitirá ejecutar una distribución de Linux. Tomaremos de ejemplo la distribución de Ubuntu 22.04 LTS.

# Habilitar subsistema de Windows para Linux (WSL)

Hay dos opciones para habilitar esta características, desde PowerShell o desde la configuración de Windows 11.

Para habilitar WSL **desde PowerShell** ejecutamos el siguiente comando:

```
wsl --install
```
Para habilitarlo desde el entorno gráfico de Windows 11 tenemos que ir a menú inicio, configuración, aplicaciones, características opcionales, más características de Windows y marcamos Subsistema de Windows para Linux.

![Activar subsistemas de Windows para Linux](/images/activar-subsistema-de-windows-para-linux.png)

> Una vez habilitado WSL será necesario aplicar un reinicio del sistema.

# Instalar Ubuntu 22.04 LTS

Para la instalación de la distribución Ubuntu 22.04 LTS, realizamos la busqueda de esta en Microsoft Store.

![Ubuntu 22.04 LTS en Microsoft Store](/images/ubuntu-22-04-microsoft-store.png)

Tras finalizar la instalación hacemos la busqueda de Ubuntu desde el menú inicio y lo ejecutamos.

![Ubuntu 22.04 en menú inicio Windows 11](/images/ubuntu-22-04-menu-inicio.png)

Se ejecutará el asistente de instalación y configuración de Ubuntu en WSL. Este proceso puede tardar unos minutos.

![Asistente de instalación de Ubuntu en WSL](/images/asistente-instalacion-ubuntu-wsl.png)

Configuramos el idioma de la distribución.

![Configuración idioma Ubuntu 22.04 LTS en WSL](/images/idioma-ubuntu-22-04-wsl.png)

Establecemos un nombre de usuario y contraseña para el inicio de sesión en Ubuntu.

![Configuración usuario y contraseña en Ubuntu 22.04 LTS en WSL](/images/usuario-contraseña-ubuntu-22-04-wsl.png)

Indicamos la ubicación de montaje y la regeneración del archivo host y resolv.conf. Aplicar la configuración que se adapte a vuestras necesidades, en este caso, para este artículo he dejado las opciones por defecto.

![Opciones de montaje, regeneración de archivo host y resolv.conf](/images/opciones-montaje-ubuntu-22-04-wsl.png)

Por último y si todo ha ido bien nos confirmará el éxito de la configuración.

![Fin configuración Ubuntu 22.04 LTS en WSL](/images/fin-configuracion-ubuntu-22-04-wsl.png)

Al volver ejecutar Ubuntu 22.04 LTS, ya podremos trabajar desde el terminal.

![Terminal Ubuntu 22.04 LTS en WSL ](/images/terminal-ubuntu-22-04-wsl.png)

# Otros comandos útiles

- Ver lista de distribuciones disponibles.

```
wsl --list --online
```
- Instalar una distribución disponible.

```
wsl --install -d <DistroName>
```

- Enumerar distribuciones instaladas con la versión WSL establecida.

```
wsl -l -v
```

- Establecer la versión predeterminada en WSL 1 o WSL 2 cuando se instala una distribución (reemplazar <Version#> por 1 o 2).

```
wsl --set-default-version <Version#>
```

- Actualizar distribución de WSL 1 a WSL 2

```
wsl --set-default-version <Version#>
```