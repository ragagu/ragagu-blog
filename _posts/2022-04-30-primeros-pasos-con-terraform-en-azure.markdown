---
layout: post
title:  Primeros pasos con Terraform y desplegando infraestructura como código en Azure
description: Vamos a instalar Terraform y a desplegar infraestructura en Azure
date:   2022-04-30 13:40:23 +0300
image:  '/images/terraform-assets.png'
tags:   [Terraform, Azure]
---

## Automatizar despliegue de la infraestructura en Azure con Terraform

El despliegue se lleva a cabo desde una máquina Windows 11 con el subsistema de Windows para Linux, haciendo uso de la distribución Ubuntu 20.04 LTS, pero, el despliegue se puede llevar a cabo desde cualquier máquina con Linux.

## Preparación del entorno para automatizar el despliegue

Primeramente, tenemos que instalar Terraform en la máquina Linux donde se va automatizar el despliegue. De forma predeterminada, Terraform no está incluido en el repositorio estándar de Ubuntu. Debemos instalar los siguientes paquetes:
```terminal
sudo apt-get install curl gnupg2 software-properties-common -y
```

Posteriormente agregamos la clave GPG Terraform y el respositorio:
```linux
sudo curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add -
```
```linux
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com 
$(lsb_release -cs) main"
```
Por último instalamos Terraform:
```linux
sudo apt-get install terraform -y
```
Para verificar la instalación de Terraform ejecutamos:
```linux
terraform -v
```

En segundo lugar, tenemos que crear un service principal para conectar el provider azurerm de Terraform con nuestro tenant de Azure. Hacemos uso de Azure CLI. Instalamos Azure CLI mediante el siguiente comando:
```linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Para crear un service principal tenemos que conectarnos al tenant de Azure con `az login`.

Con el siguiente comando creamos el service principal:
```linux
az ad sp create-for-rbac --role="Contributor"
```

Obtenemos una salida similar a la siguiente, donde obtendremos el appid, nombre, password y tenant id:
```linux
rafa@PCP-RAGAGU:~$ az ad sp create-for-rbac --role="Contributor"
In a future release, --scopes argument will become required for creating a role assignment. Please explicitly 
specify --scopes.
Creating 'Contributor' role assignment under scope '/subscriptions/c9f11e84-d381-46cd-82be4f39add24081'
The output includes credentials that you must protect. Be sure that you do not include these credentials in 
your code or check the credentials into your source control. For more information, see 
https://aka.ms/azadsp-cli
{
 "appId": "62a71c17-03f2-4234-9795-530e34a05fed",
Rafael Galante Gutiérrez
Repo: https://github.com/ragagu/cp2devopsunir
 "displayName": "azure-cli-2022-03-06-14-47-39",
 "password": "rvFpBmn_3KPOtZv.jzoK9Ox3H-QvOw~sfY",
 "tenant": "899789dc-202f-44b4-8472-a6d40f9eb440"
}
```
## Creación de estructura de directorio y archivos Terraform

Clonamos el siguiente repositorio en la máquina donde vamos automatizar el despliegue de la infraestructura con Terrafrom `git clone https://github.com/ragagu/cp2devopsunir`

> Para instalar Git ejecuta el comando `sudo apt install git`

Accedemos al directorio cp2devopsunir/terraform y vemos la estructura de archivos siguientes: