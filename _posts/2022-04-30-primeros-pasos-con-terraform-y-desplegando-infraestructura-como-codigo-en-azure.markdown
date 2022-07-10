---
layout: post
title:  Primeros pasos con Terraform y desplegando infraestructura como código en Azure
description: Vamos a instalar Terraform y a desplegar infraestructura en Azure
date:   2022-04-30 13:40:23 +0300
image:  '/images/terraform-assets.png'
tags:   [Terraform, Azure]
---

En el siguiente lab voy a instalar y configurar **Terraform** en una máquina Windows 11 con **WSL**, haciendo uso de la distribución **Ubuntu 22.04 LTS**, pero, el despliegue se puede llevar a cabo desde cualquier máquina con Linux. Posteriormente voy a desplegar la siguiente infraestructura en **Azure**:

- Un grupo de recursos.
- Una máquina virtual con 2vCPU, 4GB de RAM y 30GB HDD con una imagen CentOS (master).
- Dos máquinas virtuales con 1vCPU, 2GB de RAM y 30GB HDD con una imagen CentOS (workers).
- Tres IPs públicas dinámicas.
- Una red virtual.
- Un grupo de seguridad de red (NSG).


# Preparación del entorno para automatizar el despliegue

**En primer lugar**, tenemos que instalar Terraform en la máquina Linux donde se va automatizar el despliegue. De forma predeterminada, Terraform no está incluido en el repositorio estándar de Ubuntu. Debemos instalar los siguientes paquetes:
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
![Ejecución de 'terraform -v'](/images/terraform-v.png)

**En segundo lugar**, tenemos que crear un service principal para conectar el provider azurerm de Terraform con nuestro tenant de Azure. Hacemos uso de **Azure CLI**. Instalamos Azure CLI mediante el siguiente comando:
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
Para crear un service principal tenemos que conectarnos al tenant de Azure con:
```
 az login
```

Con el siguiente comando creamos el service principal:
```
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/529ff35f-375b-xxxx-xxxx-xxxxxxxxxxxx"
```
![Creación de un service principal](/images/creacion-service-principal.png)

## Creación de estructura de directorio y archivos Terraform

Clonamos el siguiente repositorio en la máquina donde vamos automatizar el despliegue de la infraestructura con Terrafrom `https://github.com/ragagu/laboratorios`

> Para instalar Git ejecuta el comando `sudo apt install git`

Accedemos al directorio laboratorios/terraform/0001 y vemos la estructura de archivos siguientes:

- main.tf
- network.tf
- security.tf
- vars.tf
- vm.tf

### Configuración de archivo vars.tf

- En este fichero declaramos todas las variables.
- Declaramos la localización para ubicar en la región donde se van a desplegar los recursos. Para ver las regiones disponibles: az account list-locations
- Declaramos la variable para el nombre de la talla de la máquina virtual máster. Para ver las tallas disponibles en la región seleccionada: az vm list-sizes --location westeurope
- Declaramos la variable para el nombre de la talla de las máquinas workers. Como las dos VM workers van a ser iguales, solamente va a cambiar el nombre, aplicamos el tipo lists(string) y los nombres de ambas VM en default. En los archivos main.tf, network.tf, security.tf y vm.tf haremos uso de count para configurar estas dos VMs.

Si estás haciendo uso de una suscripción de educación, están limitadas a un máximo de 4 Cores, este límite se puede ampliar, pero no es posible hacerlo con este tipo de suscripción, la solución es pasar a una suscripción de pago por uso.

Contenido archivo vars.tf:
```
# Variable aplicable a todos los recursos

variable "location" {
  type = string
  description = "Región donde se crearán los recursos de Azure"
  default = "West Europe"
}

# Variable aplicable solo a la VM master

variable "vm_size_master" {
  type = string
  description = "Talla para la VM master"
  default = "Standard_B2s" # 2vCPU, 4GB RAM
}

# Variables aplicables a las VMs worker01 y worker02

variable "vm_size_workers" {
  type = string
  description = "Talla de la VM"
  default = "Standard_B1ms" # 1vCPU, 2GB RAM
}

variable "workers" {
  type = list(string)
  description = "Despliegue de VMs worker01 y worker 02"
  default = ["worker01", "worker02"]
}
```

### Configuración de archivo main.tf

- Configuramos el provider del que hará uso Terraform, en este caso azurerm.
- Incluimos la información del service principal creado.
- Creamos y configuramos un grupo de recursos.
- Creamos y configuramos una cuenta de almacenamiento.

Como podemos ver para la configuración location se hace llamamiento a la variable var.location, de esta forma se configuraría West Europe, tal y como está definido en vars.tf. También observamos que en todos los recursos hay que indicar el nombre del grupo de recursos donde se va a desplegar, como el nombre ya está definido en la creación del grupo de recursos, el llamamiento al nombre de este en la creación de otros recursos se establece de la siguiente forma azurerm_resource_group.rg.name.

Contenido archivo main.tf:

```
# Definición provider azurerm (Azure) y versión

terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "3.13.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = "c9f11e84-e410-46cd-82xe-xxxxxxxxxxxx"
  client_id = "62p71c17-03q2-4234-9795-xxxxxxxxxxxx"
  client_secret = "rvFpTmn_3KCOtZv.jzoK9Ox3H-xxxxxxxx"
  tenant_id = "899789xd-202f-44b4-9583-xxxxxxxxxxxx"
}

# Creación de grupo de recursos

resource "azurerm_resource_group" "rg" {
    name = "lab_0001"
    location = var.location

    tags = {
        environment = "lab_0001"
    }
}

# Creación de cuenta de almacenamiento

resource "azurerm_storage_account" "stAccount" {
    name = "salab0001"
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    account_tier = "Standard"
    account_replication_type = "LRS"

    tags = {
        environment = "lab_0001"
    }

}
```

### Configuración del archivo network.tf

- Definimos una vnet: 10.0.0.0/16
- Definimos una subnet: 10.0.1.0/24
- La subnet debe estar contenida dentro de la vnet.
- La NIC de todas las VMs deben estar contenidas dentro del mismo espacio de direcciones de la subnet.
- Asignamos un IP privada estática a las VMs.
- Creamos una IP pública dinámica para poder acceder desde fuera de Azure.

La asignación de una IP pública estática incrementa el coste de este recurso, sin embargo, la asignación de una IP privada estática es gratuita. Al configurar la IP pública dinámica, con el encendido y apagado de las VMs la dirección puede cambiar. En la creación de los workers en el nombre del recurso se establece -${var.workers[count.index]} para que automáticamente coja del fichero vars.tf de la variable workers los nombres definidos en default. Para que eso funcione se declara también en el recurso a crear count = length(var.workers). Como a la VM master le hemos dado la IP 10.0.1.10, para que los workers se asignen automáticamente la primera IP privada libre a partir de las 10.0.1.11 para que no se duplique la IP del master, aplicamos la siguiente configuración private_ip_address = "10.0.1.${count.index + 11}".

Contenido de archivo network.tf:

```
# Creación de red virtual

resource "azurerm_virtual_network" "myNet" {
    name = "lab-0001-vnet"
    address_space = ["10.0.0.0/16"]
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name

    tags = {
        environment = "lab-0001"
    }
}

# Creación de subnet

resource "azurerm_subnet" "mySubnet" {
    name = "subnet-lab-0001"
    resource_group_name = azurerm_resource_group.rg.name
    virtual_network_name = azurerm_virtual_network.myNet.name
    address_prefixes = ["10.0.1.0/24"]
}

# Creación de interfaz de red para VM master

resource "azurerm_network_interface" "nicmaster" {
    name = "nic-master"
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name

      ip_configuration {
      name = "ipconf-master"
      subnet_id = azurerm_subnet.mySubnet.id
      private_ip_address_allocation = "Static"
      private_ip_address = "10.0.1.10"
      public_ip_address_id = azurerm_public_ip.myPublicIpMaster.id
}

      tags = {
          environment = "lab-0001"
      }
}

# Creación de interfaces de red para VMs worker01 y worker02

resource "azurerm_network_interface" "nicworkers" {
    name = "nic-${var.workers[count.index]}"
    count = length(var.workers)
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name

      ip_configuration {
      name = "ipconf-${var.workers[count.index]}"
      subnet_id = azurerm_subnet.mySubnet.id
      private_ip_address_allocation = "Static"
      private_ip_address = "10.0.1.${count.index + 11}"
      public_ip_address_id = azurerm_public_ip.myPublicIpWorkers[count.index].id
}

      tags = {
          environment = "lab-0001"
      }
}

# IP pública VM master

resource "azurerm_public_ip" "myPublicIpMaster" {
    name = "publicip-master"
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name
    allocation_method = "Dynamic"
    sku = "Basic"

      tags = {
          environment = "lab-lab0001"
      }
}

# IP pública VMs worker01 y worker02

resource "azurerm_public_ip" "myPublicIpWorkers" {
    name = "publicip-${var.workers[count.index]}"
    count = length(var.workers)
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name
    allocation_method = "Dynamic"
    sku = "Basic"

      tags = {
          environment = "lab-0001"
      }
}
```

### Configuración del archivo security.tf

- Creamos un grupo de seguridad de red (NSG).
- En el NSG definimos el tráfico que vamos a autorizar incluyendo reglas de seguridad. De momento solo se va a permitir la regla de entrada por SSH.
- Asociamos el NSG con todas las NIC de las VMs que se van a desplegar. De esta forma la regla que creamos se aplica a todas las VMs.

Contenido del archivo security.tf:

```
# Grupo de seguridad de red

resource "azurerm_network_security_group" "mySecGroup" {
    name = "nsg-k8s"
    location = azurerm_resource_group.rg.location
    resource_group_name = azurerm_resource_group.rg.name

    security_rule {
        name = "SSH"
        priority = "1001"
        direction ="Inbound"
        access = "Allow"
        protocol = "Tcp"
        source_port_range = "*"
        destination_port_range = "22"
        source_address_prefix = "*"
        destination_address_prefix = "*"
    }

    tags = {
        environment = "lab-0001"
    }
}

# Vincular grupo de seguridad de red con interfaz de red de vm master

resource "azurerm_network_interface_security_group_association" "associationmaster" {
    network_interface_id = azurerm_network_interface.nicmaster.id
    network_security_group_id = azurerm_network_security_group.mySecGroup.id
}


# Vincular grupo de seguridad de red con interfaz de red de worker01 y worker02

resource "azurerm_network_interface_security_group_association" "associationworers" {
    count = length(var.workers)
    network_interface_id = azurerm_network_interface.nicworkers[count.index].id
    network_security_group_id = azurerm_network_security_group.mySecGroup.id
}
```

### Configuración del archivo vm.tf

- Definimos la creación de la VM master.
- Definimos la creación de las VMs workers.
- Definimos el tamaño.
- Le asignamos las NICs creadas en network.tf.
- Indicamos el usuario administrador.
- Especificamos la clave pública para el usuario administrador. Para crear una clave SSH, para poder conectarnos a la VM hacemos uso del siguiente comando ssh-keygen -t rsa -b 4096. Por defecto la clave se almacena en ~/.ssh, el nombre que le hemos asignado es id_rsa_cp2_devops porque ya disponía de una clave con el nombre por defecto (id_rsa). No hemos asignado una contraseña a la clave para esta práctica. Una vez dispongamos de la clave SSH hay que restablecer la ruta de la misma en el fichero vm.tf.
- Utilizaremos el usuario especificado y la clave privada asociada a la pública para acceder a la VM.
- Definimos el tipo de disco y la replicación (Standar_LRS).
- Cuando definamos images del Marketplace tendremos que definir plan y source_image_reference con los datos de la imagen que utilizaremos.
- Definimos la storage account a utilizar para almacenar información de troubleshooting.

Para definir la imagen a desplegar del Marketplace, para ver todas las disponibles con CentOS ejecutamos el siguiente comando:
```
az vm image list --offer CentOS --all --output table
```
Desde la salida que obtenemos en formato tabla podemos elegir la imagen y tendremos otros datos necesarios para incluir en vm.tf como son offer, publisher, version, name, plan, sku y product. Una vez seleccionada una imagen, tendremos que aceptar los términos de uso de esta, si no, porque desde Terraform no es posible aceptar esos términos y condiciones. Para esta práctica se ha elefido la siguiente imagen:

name = "centos-8-3-free"
product = "centos-8-3-free"
publisher = "cognosys"
offer = "centos-8-3-free"
sku = "centos-8-3-free"
version = "1.2019.0810"

Por lo tanto, el comando a ejecutar para aceptar los términos y condiciones es:
```
az vm image terms accept --offer centos-8-3-free --plan 1.2019.0810 --publisher cognosys
```
Contenido del archivo vm.tf:

```
# Creación de máquina master

resource "azurerm_linux_virtual_machine" "vmmaster" {
    name = "master"
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    size = var.vm_size_master
    admin_username = "rafadmin"
    network_interface_ids = [ azurerm_network_interface.nicmaster.id ]
    disable_password_authentication = true
    admin_ssh_key {
        username = "rafadmin"
        public_key = file ("~/.ssh/id_rsa_cp2_devops.pub")
    }


    os_disk {
        caching = "ReadWrite"
        storage_account_type = "Standard_LRS"
    }

    plan {
        name = "centos-8-3-free"
        product = "centos-8-3-free"
        publisher = "cognosys"
    }
  
      source_image_reference {
        publisher = "cognosys"
        offer = "centos-8-3-free"
        sku = "centos-8-3-free"
        version = "1.2019.0810"
    }

    boot_diagnostics {
        storage_account_uri = azurerm_storage_account.stAccount.primary_blob_endpoint
    }

    tags = {
        environment = "lab-0001"
    }

}


# Creación de máquinas virtuales worker01 y worker02

resource "azurerm_linux_virtual_machine" "vmsworkers" {
    name = "${var.workers[count.index]}"
    count = length(var.workers)
    resource_group_name = azurerm_resource_group.rg.name
    location = azurerm_resource_group.rg.location
    size = var.vm_size_workers
    admin_username = "rafadmin"
    network_interface_ids = [ azurerm_network_interface.nicworkers[count.index].id ]
    disable_password_authentication = true

    admin_ssh_key {
        username = "rafadmin"
        public_key = file ("~/.ssh/id_rsa_cp2_devops.pub")
    }

    os_disk {
        caching = "ReadWrite"
        storage_account_type = "Standard_LRS"
    }

    plan {
        name = "centos-8-3-free"
        product = "centos-8-3-free"
        publisher = "cognosys"
          }

    source_image_reference {
        publisher = "cognosys"
        offer = "centos-8-3-free"
        sku = "centos-8-3-free"
        version = "1.2019.0810"
    }

    boot_diagnostics {
        storage_account_uri = azurerm_storage_account.stAccount.primary_blob_endpoint
    }

    tags = {
        environment = "lab-0001"
    }
}
```

# Despliegue automatizado de la infraestructura con Terraform

Una vez configuramos todos los archivos .tf es hora de preparar, planear y aplicar.

1. Preparar la infraestructura

La primera vez que se va a realizar un despliegue o tras aplicar cambios en los archivos .tf debemos inicializar la configuración de Terraform, para ello ejecutamos el siguiente comando:
```
terraform init
```
Si todo ha ido correctamente debemos obtener el siguiente resultado:

![Ejecución de 'terraform init'](/images/terraform-init.png)

2. Realizar un plan

El segundo paso es realizar un terraform plan. Esta funcionalidad nos permite ver las acciones que se van a realizar para lograr el estado deseado, para ello ejecutamos el siguiente comando:
```
terraform plan
```
Debemos obtener una salida sin errores, donde nos detalle que recursos vamos añadir, actualizar o destruir.

Si observamos toda la salida, nos vendrá marcada con un + sobre todos los reucrsos que se van a crear. Esto puede variar cuando realicemos un cambio a futuro, si un recurso se va a destruir viene indicado con un -, si se va actualizar con un ~. De esta forma nos permite ver las acciones que se van a llevar a cabo antes del despligue.

![Ejecución de 'terraform plan'](/images/terraform-plan.png)

3. Aplicar el despliegue

Por último, para aplicar el despliegue tendremos que ejecutar el comando 
```
terraform apply
```
Este comando ejecutará primero un plan, que posteriormente tendremos que aceptar con un 'yes' o 'no' para realizar o no el despliegue.

![Ejecución de 'terraform apply'](/images/terraform-apply.png)

Durante el despliegue podemos ver la salida de los recursos que se están creado. Tras la finalización podremos observar que recursos se han agregado, actualizado o destruido. En este caso solo se deben crear recursos.

![Salida obtenida tras aplicar 'terraform apply'](/images/terraform-apply-output.png)

Terraform almacena el estado de la infraestructura en un archivo llamado terraform.state. De esta forma Terraform detecta cuando se producen cambios en la infraestructura, compara los archivos .tf con el .state. Si se agregan nuevos recursos a los ficheros .tf detectará que no existen en .state y los creará, si se elimina algún recursos de los archivos .tf detectará que en .state está y los eliminará y por último si se realiza una acutalización de un recurso en el archivo .tf por ejemplo cambiar una IP de estática a dinámica, Terraform detectará que en el .state existe la IP como estática y aplicará el cambio a dinámica.

Tras realizar el despliegue podemos comprobar en Azure los recursos que se han creado:

![Creación grupo de recursos lab_0001](/images/grupo-de-recursos-lab0001.png)

![Recursos lab_0001](/images/recursos-lab0001.png)

![Máquinas virtuales lab_0001](/images/vms-lab0001.png)

Para destruir o eliminar la infraestructura creada tenemos que ejecutar el siguiente comando y confirmar con 'yes':

```
terraform destroy
```
![Ejecución comando 'terraform destroy'](/images/terraform-destroy.png)

Tras confirmar la eliminación obtendremos la siguiente salida:

![Salida comando 'terraform destroy'](/images/destroy-output.png)


Fin del laboratorio, cualquier duda pueden dejar sus comentarios.