---
layout: post
title: "Cómo configurar PyCharm para compilar los archivos SASS"
date: 2015-06-29 10:10:03 +0200
comments: true
categories: PyCharm
---

Suponemos que tenemos un proyecto con las siguientes carpetas:

    app/
       static/
            css/
            sass/
                main.sass
                home.sass
                variables.sass

y queremos que cada vez cuando editemos alguno de los archivos ``.sass``, PyCharm lo compile y ponga en la carpeta ``app/static/css/``:

    app/
       static/
            css/
                main.css
                home.css
            sass/
                main.sass
                home.sass
                variables.sass

Para esto debemos ir a ``File -> Settings -> File Watchers`` y agregar un nuevo watcher, escogiendo la opción ``SASS``

{% img /images/pycharm_file_watchers.png %}

y editar las configuraciones:

{% img /images/pycharm_file_watchers_settings.png %}

