---
layout: post
title: "Cómo hacer bot para Facebook en Python"
date: 2017-02-04 11:27:15 -0500
comments: true
published: true
categories: 
- Facebook
- Python
- Bot
- Messenger
---

Primero creamos una aplicación en Facebook, vamos al menú de configuración y agregamos el servicio de messenger ("Add Product"):

{% img center /images/bots/fb_bot_webhook.png %}

<!-- more -->

Luedo vamos a hacer click en el botón "Setup Webhooks":

{% img center /images/bots/fb_bot_register.png %}

Para poder verificar que nuestro bot esté bien configurado, Facebook nos mandará una peticicón GET a nuestro servidor

```
GET https://mipagina.com/bot/fb-webhook/?hub.mode=subscribe&hub.challenge=1122334455&hub.verify_token=una-frase-secreta
```

y esperará que le respondemos con el código 200 y el contenido del paramentro `hub.challenge` cuando `verify_token` coincide con la frase secreta que especificamos, o con el código HTTP 403 en el caso opuesto.

Entonces esribimos una vista que se encargará de eso:

{% codeblock urls.py lang:python %}
    # ...
    url(
        r'fb-webhook/$',
        FacebookCommandReceiveView.as_view(),
        name='fb_webhook',
    ),
    # ...
{% endcodeblock %}

{% codeblock urls.py lang:python %}
class FacebookCommandReceiveView(View):

    def get(self, request):
        if request.GET.get('hub.mode') == 'subscribe' and request.GET.get('hub.verify_token') == settings.FACEBOOK_VERIFY_TOKEN:
            return HttpResponse(request.GET.get('hub.challenge'))
        else:
            return HttpResponseForbidden()

{% endcodeblock %}

Cuando la vista esté lista y desplegada en producción, podemos oprimir el botón "Verify and Save". Después de eso Facebook nos dará un token de la aplicación y permitirá configurar un *webhook*:

{% img center /images/bots/fb_bot_settings.png %}

Ahora podemos suscribirnos a los eventos en messenger, es decir: cada vez que alguien escibe un mensaje vía messanger a nuestra página de Facebook, Facebook enviará una petición POST al weebhook con el JSON que tendrá todos los datos del mensaje.

{% img center /images/bots/fb_bot_add_subscription.png %}

Desúés de oprimir el botón "Add to Subscription", Facebook nos mostrará el enlace "Edit notes":

{% img center /images/bots/fb_bot_current_subscription.png %}

Damos click en "Edit notes" y entramos a el formulario de configuración. Aquí especificamos la página a los mensajes de la cual vamos a suscribirnos e escribimos ejemplos de respuestas que dará nuesto bot a ciertos mensajes en Messanger:

{% img center /images/bots/fb_bot_edit_subscription.png %}

Eso sirve para que Facebook pueda verificar que nuestro bot efectivamente responde bien a los mensajes de chat.

Después de que Facebook verifique el comportamiento de nuestro bot, recibimos el mensaje de aprobación, diciendo que desde ahora nuestro bot está activo:

{% img center /images/bots/fb_bot_subscription_status.png %}

Eso nos permite hacer pruebas de la plataforma, porque el bot va a poder responder sólo a los mensajer enviados por los administradores de nuestra página de Facebook.

Cuando nuestro bot esté listo, lo podemos activar para todos los usuarios (hacerlo público):

{% img center /images/bots/fb_bot_publish.png %}


Enlaces:

* [Quick Start](https://developers.facebook.com/docs/messenger-platform/guides/quick-start): cómo crear y configurar webhooks.