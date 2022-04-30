---
layout: post
title:  Primeros pasos con Terraform y desplegando infraestructura como código en Azure
description: Vamos a instalar Terraform y a desplegar infraestructura en Azure
date:   2022-04-30 13:40:23 +0300
image:  '/images/terraform-assets.png'
tags:   [Terraform, Azure]
---

En el siguiente lab voy a instalar y configurar Terraform en una máquina Windows 11 con WSL, haciendo uso de la distribución Ubuntu 20.04 LTS, pero, el despliegue se puede llevar a cabo desde cualquier máquinas con Linux. Posteriormente voy a desplegar la siguiente infraestructura en Azure:

- Un grupo de recursos.
- Una máquina virtual con 2vCPU, 4GB de RAM y 30GB HDD con una imagen CentOS.
- Dos máquinas virtuales con 1vCPU, 2GB de RAM y 30GB HDD con una imagen CentOS.
- Tres IPs públicas dinámicas.
- Una red virtual.
- Un grupo de seguridad de red.


## Preparación del entorno para automatizar el despliegue

En primer lugar, tenemos que instalar Terraform en la máquina Linux donde se va automatizar el despliegue. De forma predeterminada, Terraform no está incluido en el repositorio estándar de Ubuntu. Debemos instalar los siguientes paquetes:
```
sudo apt-get install curl gnupg2 software-properties-common -y
```
Posteriormente agregamos la clave GPG Terraform y el respositorio:
```
sudo curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add -
```
```
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com 
$(lsb_release -cs) main"
```
Por último instalamos Terraform:
```
sudo apt-get install terraform -y
```
Para verificar la instalación de Terraform ejecutamos:
```
terraform -v
```

En segundo lugar, tenemos que crear un service principal para conectar el provider azurerm de Terraform con nuestro tenant de Azure. Hacemos uso de Azure CLI. Instalamos Azure CLI mediante el siguiente comando:
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
Para crear un service principal tenemos que conectarnos al tenant de Azure con `az login`.

Con el siguiente comando creamos el service principal:
```
az ad sp create-for-rbac --role="Contributor"
```
Obtenemos una salida similar a la siguiente, donde obtendremos el appid, nombre, password y tenant id:
```
In a future release, --scopes argument will become required for creating a role assignment. Please explicitly 
specify --scopes.
Creating 'Contributor' role assignment under scope '/subscriptions/c9f11e84-d381-46cd-82be4f39add24081'
The output includes credentials that you must protect. Be sure that you do not include these credentials in 
your code or check the credentials into your source control. For more information, see 
https://aka.ms/azadsp-cli
{
 "appId": "62p71c17-03q2-4234-9795-xxxxxxxxxxxx",
 "displayName": "azure-cli-2022-03-06-14-47-39",
 "password": "rvFpTmn_3KCOtZv.jzoK9Ox3H-xxxxxxxx",
 "tenant": "899789xd-202f-44b4-9583-xxxxxxxxxxxx"
}
```
## Creación de estructura de directorio y archivos Terraform

Clonamos el siguiente repositorio en la máquina donde vamos automatizar el despliegue de la infraestructura con Terrafrom `https://github.com/ragagu/laboratorios`

> Para instalar Git ejecuta el comando `sudo apt install git`

Accedemos al directorio cp2devopsunir/terraform y vemos la estructura de archivos siguientes:

- main.tf
- network.tf
- security.tf
- vars.tf
- vm.tf