---
layout: post
title:  Activar/desactivar la autenticación SMTP en cuentas de Exchange Online
description: Se recomienda habilitar la autenticación SMTP solo para las cuentas o buzones que lo requieran...
date:   2022-03-19 15:01:35 +0300
image:  '/images/07.jpg'
tags:   [Microsoft 365, Exchange Online]
---
El protocolo SMTP se usa en los siguientes escenarios en Microsoft 365:

* Clientes POP3 e IMAP4
* Aplicaciones, servidores de informes y dispositivos multifunción que generan y envían mensajes de correo electrónico.

El puerto que se usa para la autenticación SMTP es TCP 587.

Se recomienda habilitar la autenticación SMTP solo para las cuentas o buzones que lo requieran dado que todos los clientes de correo eletrónico modernos que se conectan a Exchange Online o Microsoft 365, como por ejemplo, Outlook, Outlook en la web, Mail de iOS, Outlook para iOS y Android, etc. no usan la autenticación SMTP para enviar mensajes de correo electrónico.

> Si ha habilitado los valores predeterminados de seguridad en su empresa, AUTH SMTP ya está deshabilitado en Exchange Online.

## Requisitos para activar o desactivar SMTP y conexión a Exchange Online mediante PowerShell

1. Tener habilitada la ejecución de scripts.

```powershell
Set-ExecutionPolicy RemoteSigned
```

2. Tener instalado el módulo de Azure AD.

```powershell
Install-Module MSOnline
```

3. Creamos un objeto de credenciales, para ello ejecutamos el siguiente comando e introducimos las credenciales de nuestro administrador global de Microsoft 365.

```powershell
$credential = Get-Credential
```

4. Importamos el módulo de Microsoft 365, para ello ejecutamos el siguiente comando.

```powershell
Import-Module MsOnline
```

5. Con el objeto de credenciales que creamos anteriormente con el módulo MSOnline, ahora nos podemos conectar a Microsoft 365 ejecutando el siguiente comando.

```powershell
Connect-MsolService -Credential $credential
```

6. Ejecutamos el siguiente comando para crear una sesión remota de Exchange Online.

```powershell
$exchangeSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://outlook.office365.com/powershell-liveid/" -Credential $credential -Authentication "Basic" –AllowRedirection
```

```powershell
Import-PSSession $exchangeSession
```

## Deshabilitar autenticación SMTP

Conectados a Exchange Online remotamente por PowerShell para deshabilitar SMTP globalmente en la compañía, ejecutamos el siguiente comando.

```powershell
Set-TransportConfig -SmtpClientAuthenticationDisabled $true
```

Para comprobar que se ha deshabilitado la autenticación SMTP ejecutamos el siguiente comando y comprobamos que el valor de la propiedad SmtpClientAuthenticationDisablede es True.

```powershell
Get-TransportConfig | Format-List SmtpClientAuthenticationDisabled
```

## Habilitar autenticación SMTP

Conectados a Exchange Online remotamente por PowerShell comprobamos el usuario, donde usuario@contoso.com introducimos la cuenta de usuario a revisar.

```powershell
Get-CASMailbox -Identity Usuario@contoso.com | Format-List SmtpClientAuthenticationDisabled
```

Si nos carga el usuario con el valor en blanco o $True ejecutaríamos el siguiente comando para habilitar la autenticación SMTP.


```powershell
Set-CASMailbox -Identity sean@contoso.com -SmtpClientAuthenticationDisabled $false 
```


Documentación de apoyo:
https://docs.microsoft.com/es-es/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission