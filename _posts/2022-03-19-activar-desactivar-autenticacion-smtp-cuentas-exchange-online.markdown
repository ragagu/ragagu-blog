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

1. Tener habilitada la ejecución de scripts

{% highlight powershell %}
Set-ExecutionPolicy RemoteSigned
{% endhighlight %}

{% highlight js %}
  $('.top').click(function () {
    $('html, body').stop().animate({ scrollTop: 0 }, 'slow', 'swing');
  });
  $(window).scroll(function () {
    if ($(this).scrollTop() > $(window).height()) {
      $('.top').addClass("top-active");
    } else {
      $('.top').removeClass("top-active");
    };
  });
{% endhighlight %}

***



