---
layout: post
title: "Inteligencia Artificial en Rusia"
date: 2013-10-12 08:57:18 -0500
comments: true
categories:
---

{% img right /images/academia.jpg 'Academia de Ciencias de Rusia' %}

Primero determinamos ¿qué es la inteligencia artificial? Según la definición popular es una rama de la computación que se dedica a la creación de los programas con objeto de simular o repetir las capacidades típicas de los seres humanos. Como dijo John McCarthy: “Es la ciencia e ingenio de hacer máquinas inteligentes, especialmente programas de cómputo inteligentes.”

Pero hay otra definición que me parece más concreta y representa mejor el concepto. Inteligencia artificial es una ciencia experimental que está dedicada a los problemas para los cuales no se conoce un algoritmo para resolverlos. Por ejemplo, cuando golpeamos una pelota y queremos saber dónde estará en un momento del tiempo particular, usamos la ecuación del movimiento para esta pelota. O cuando necesitamos ordenar un conjunto de números, simplemente usamos el método de ordenamiento de burbuja o cualquier otro, que ya sabemos de antemano.

Pero hay una clase de tareas que no son tan obvias y requieren unos métodos más sofisticados para resolverlas.

El primer ejemplo de uso de inteligencia artificial es la resolución de las tareas médicas. En el Centro Nacional de Gerontología ya usan el sistema que ayuda dar diagnósticos de algunas enfermedades. Obviamente, no tienen como fin substituir al médico, pero ayudan a prevenir los errores humanos: el doctor puede estar cansado o tener mucho trabajo repetitivo y por eso puede cometer errores mecánicos. Por eso usan el sistema que de forma paralela con el doctor hace el mismo análisis, luego compara los resultados y si ellos no coinciden, advierte al doctor para que revise su trabajo.

Por ejemplo, fue desarrollado un módulo que analiza las imágenes de las películas, es decir, los fragmentos deshidratados de los líquidos biológicos, y determina el nivel de la enfermedad del paciente.

En la primera etapa, se toma una foto de la película, y después del procesamiento digital de las imágenes, se obtiene el vector de las características. Luego lo envían a la red neuronal o el método de grupos para manejo de datos (capacitados de antemano con los datos de entrenamiento), que devuelven la nivel de la enfermedad.
En el Instituto de Sistemas de Programas, que está basado en la ciudad de Pereslavl-Zalessky, fue desarrollado el sistema que se llama INTERIN. Este sistema contiene unos módulos que sirven para ayudar a manejar un hospital: desde el registro de los pacientes hasta los módulos de diagnóstico y sistemas expertos. Los programas de INTERIN ya están instalados en más de 70 hospitales en Rusia.

Otro campo muy importante donde usan los métodos de inteligencia artificial es el programa espacial ruso. Con el apoyo de la agencia espacial rusa Roscosmos se realizan muchos proyectos.

A uno de ellos se llama el proyecto de “Encuentro espacial”. Es un sistema complejo que contiene módulos para el reconocimiento de las imágenes (que ayuda encontrar los marcadores en la nave espacial), el módulo de visualización de información (que ayuda al operador a manejar la nave) y otros que ayudan a prevenir los errores.

Otra área es la detección de los casos de emergencia. Por ejemplo, la comunicación entre la nave espacial y la estación base en la Tierra se caracteriza por tener muchos parámetros diferentes (como pérdida de la conexión y otros).

Todos estos parámetros se pueden escribir en un vector que determina el estado del sistema en un momento de tiempo particular. Enviamos cada vector al sistema de reconocimiento de los patrones, entrenado de antemano, y luego recibimos el estado de la situación. Si ese estado coincide con uno de los estados de emergencia, el programa advierte al operador.

Otro campo, muy interesante es el de los sistemas dinámicos. Son los sistemas que usan la lógica de predicados con que construyen los algoritmos de la toma de decisiones. Usando este método fue desarrollado un algoritmo para regular el movimiento de los carros que atraviesan el paso semaforizado. Basándose en esa lógica se creó un sistema de reglas que describe las situaciones posibles que pueden ocurrir.

Usando el mismo principio en el Instituto Ruso de Problemas en Inteligencia Artificial han desarrollado un sistema experto que se dedica a la toma de decisiones. Ese programa no depende del campo al que lo aplican, simplemente necesita uno o más expertos que tienen que responder algunas preguntas sobre el campo de aplicación. Luego sus respuestas se ponen en la base de datos de conocimiento, que luego se puede usar para obtener la información buscada.
