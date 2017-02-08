---
layout: post
title: "Cómo hacer bot para Facebook en Python"
date: 2017-02-04 11:27:15 -0500
comments: true
categories: 
- Facebook
- Python
- Bot
- Messenger
---

Primero creamos una aplicación en Facebook, vamos al menú de configuración y agregamos el servicio de messenger ("Add Product"):

{% img center /images/bots/fb_bot_webhook.png %}

Luedo vamos a hacer click en el botón "Setup Webhooks":

{% img center /images/bots/fb_bot_register.png %}

Para poder verificar que nuestro bot esté bien configurado, Facebook nos mandará una peticicón GET a nuestro servidor

```
GET https://mipagina.com/bot/fb-webhook/?hub.mode=subscribe&hub.challenge=1122334455&hub.verify_token=una-frase-secreta
```

y esperará que le respondemos con el código 200 y el contenido del paramentro `hub.challenge` cuando `verify_token` coincide con la frase secreta que especificamos, o con el código HTTP 403 en el caso opuesto.

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

Después de oprimir el botón "Verify and Save"

{% img center /images/bots/fb_bot_settings.png %}

podemos generar *Access Token* para nuestra página y suscribirnos a los eventos en messenger.

{% img center /images/bots/fb_bot_add_subscription.png %}

{% img center /images/bots/fb_bot_current_subscription.png %}

{% img center /images/bots/fb_bot_edit_subscription.png %}

{% img center /images/bots/fb_bot_subscription_status.png %}

{% img center /images/bots/fb_bot_publish.png %}


Enlaces:

* [Quick Start](https://developers.facebook.com/docs/messenger-platform/guides/quick-start): cómo crear y configurar webhooks.