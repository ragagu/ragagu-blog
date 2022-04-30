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
```ubuntu
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com 
$(lsb_release -cs) main"
```
Por último instalamos Terraform:
```ubuntu
sudo apt-get install terraform -y
```
