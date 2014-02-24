---
layout: post
permalink: guias/ejecutando-comandos-en-paralelo
title: Ejecutando comandos en paralelo
categories: [guía, hacking, productividad, tecnología]
tags:
---
En estos días casi todos los dispositivos electrónicos que utilizamos (computadores, portátiles, celulares) vienen con capacidades de multiprocesamiento y como tecnólogo me es útil saber una forma simple y fácil de utilizar estos múltiples procesadores. Sé que puedo utilizar lenguajes de programación para distribuir múltiples tareas entre varios procesadores, pero eso requiere escribir código. Buscando una forma aún más “fácil” que programar, encontré una interesante herramienta: [**GNU parallel**][sitio].

Existe una [guía oficial][guía] explicando la mayoría de las características de esta herramienta. Esa guía es bastante extensa pero lamentablemente no es muy amigable. Entonces decidí escribir esta pequeña guía amigable con la intención de que ayude a entender las características básicas de esta poderosa herramienta.

![GNU parallel](/images/ejecutando-comandos-en-paralelo/gnu-parallel.png "GNU parallel")

# ¿Cómo instalar?

## Ubuntu

    sudo apt-get install parallel

## OSX

    brew install parallel

Después de la instalación tendremos acceso a un ejecutable: `parallel`. Para verificar que está instalado correctamente podemos ejecutar `parallel --version`, este comando mostrará una respuesta parecida a esta:

    GNU parallel 20131222
    Copyright (C) 2007,2008,2009,2010,2011,2012,2013 Ole Tange and Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    GNU parallel comes with no warranty.

    Web site: http://www.gnu.org/software/parallel

    When using GNU Parallel for a publication please cite:

    O. Tange (2011): GNU Parallel - The Command-Line Power Tool,
    ;login: The USENIX Magazine, February 2011:42-47.

Ahora que ya tenemos instalada la herramienta podemos comenzar.

# Uso simple

Bueno, primero vamos intentar ejecutar dos comandos en paralelo utilizando **GNU parallel**:

    parallel ::: foo bar

En este caso los comandos `foo` y `bar` son enviados como parámetros para `parallel`. **GNU parallel** ejecuta los dos comandos en paralelo utilizando tantos procesadores tenga la máquina y luego concatena la respuesta de ambos comandos y la muestra en la Salida estándar *(stdout)*. Los `:::` son importantes para diferenciar entre las opciones que **GNU parallel** puede recibir y entre los comandos a ejecutar en paralelo.

Es útil ejecutar diferentes comandos en paralelo, pero muchas veces queremos ejecutar un mismo comando en paralelo pero enviando diferentes parámetros. Con lo que ya sabemos de **GNU parallel** podríamos intentar lo siguiente:

    parallel ::: 'make sandwich' 'make café'

Como podemos ver, tenemos que repetir el comando `make` y lo único que cambia son los parámetros `sandwich` y `café`. Para evitar esa duplicación podemos decirle a `parallel` que ejecute un mismo comando y le pasamos los parámetros para ese comando:

    parallel make ::: sandwich café

En este caso el comando `make` es ejecutado en paralelo con el parámetro `sandwich` y también con el otro parámetro `café`.

Una forma para evaluar cómo es que **GNU parallel** ejecutará los comandos es pasandolé el parámetro `dry-run`, que es mas o menos un simulacro, es decir solo mostrará en pantalla las instrucciones que se ejecutarán en paralelo, pero no las ejecutará *(solo simulará)*. Podemos simular el anterior ejemplo de la siguiente manera:

    parallel --dry-run make ::: sandwich café

La salida sería la siguiente:

    make sandwich
    make café

Ahora veamos un ejemplo *real* para poner en práctica lo visto hasta ahora.

# Ejemplo: Iniciando servicios en paralelo

En el proyecto que estas trabajando actualmente tienen algunas pruebas automatizadas. Antes de ejecutar las pruebas se tiene que iniciar unos servicios **REST**, ya que tu **SUT** (Sistema bajo prueba) depende de estos.

Para iniciar las pruebas automatizadas tienes un comando bastante simple e intuitivo `pruebas_automatizadas.sh`. Para iniciar los servicios **REST** tienes el comando `iniciar_servicio.sh`, este comando recibe como parámetro el nombre del servicio que se quiere iniciar. Los nombres de los servicios son: `sol`, `luna`, `ponce`, `fraile`, `bennett` [^1].

Cada servicio demora 10 segundos para iniciar [^2] pero el problema es que tienes que iniciar todos los servicios antes de ejecutar las pruebas automatizadas. Esto significa que antes de ejecutar las pruebas tienes que esperar **50 segundos**, que es el tiempo minimo hasta que todos los servicios estén listos para ser utilizados.

Lo que tienes ahora en tu proyecto es un comando `pruebas.sh` que se ve algo asi:

    # pruebas.sh

    ./iniciar_servicio.sh sol
    ./iniciar_servicio.sh luna
    ./iniciar_servicio.sh ponce
    ./iniciar_servicio.sh fraile
    ./iniciar_servicio.sh bennett

    ./pruebas_automatizadas.sh

Ahora que ya conoces la herramienta **GNU parallel** decides utilizarla para ganar un poco de tiempo, decides ejecutar en paralelo la parte de iniciar los servicios. Una nueva versión del comando `pruebas.sh` queda así:

    # pruebas.sh

    parallel ./iniciar_servicio.sh ::: sol luna ponce fraile bennett

    ./pruebas_automatizadas.sh

Con esto pudiste reducir de **50 segundos a 10 segundos** la parte de inicialización de servicios, eso significa que tienes una respuesta *(feedback)* más rápida al ejecutar las pruebas automatizadas, ¡Jallalla! [^3].

# Uso más avanzado

## Entrada mediante archivo

Los comandos que se pasan a **GNU parallel** para que sean ejecutados en paralelo pueden estar almacenados en un archivo. Por ejemplo en el archivo `comandos.txt` podemos tener lo siguiente:

    foo
    bar

Y para que **GNU parallel** reciba sus argumentos de ese archivo, ejecutamos lo siguiente:

    parallel -a comandos.txt

## Entrada mediante "Entrada estándar"

**GNU parallel** tambien puede recibir los comandos por la Entrada estándar *(stdin)*. Usando el archivo `comandos.txt` del ejemplo anterior, podemos redireccionar su contenido para la Entrada estándar para luego procesar los comandos en paralelo, de la siguiente manera:

    cat comandos.txt | parallel

## Manipulación de cadenas de caracteres

Un caso común es enviar a **GNU parallel** como parámetros rutas de archivos, en general porque queremos procesarlos en paralelo, para eso tenemos algunas formas de tratar estos parámetros.

Para hacer referencia al parámetro que **GNU parallel** recibe utilizamos las llaves `{}`.

Como dije, en general se pasan rutas de archivos como parámetros. Una transformación que es útil es la de remover la extensión del archivo, esto se logra usando un punto dentro de las llaves `{.}`.

Veamos otro ejemplo "real" para utilizar lo estudiado hasta ahora.

# Ejemplo: Convertir formato de tu colección de música

Tienes toda tu colección de música en la carpeta `/música`, toda tu colección consta de miles de archivos en formato **MP3**. Después de haber asistido a una convención de **Software Libre** y haber escuchado acerca de formatos abiertos, decidiste que vas a convertir toda música para el formato **Ogg** [^4].

Para para convertir la música decides utilizar la herramienta [**ffmpeg**][ffmpeg]. Después de leer la documetación sabes que para convertir un archivo de un formato a otro solo tienes que especificar el nombre del nuevo archivo con su nueva extensión y **ffmepg** hará el resto del trabajo. Por ejemplo para convertir tu música favorita de **MP3** a **Ogg** ejecutas lo siguiente:

    ffmpeg -i favorita.mp3 favorita.ogg

Ahora que sabes cómo convertir un archivo, te decides a convertir toda tu colección, pero sabes que hacerlo de la manera tradicional *(en secuencia)* va a ser un proceso que tomará mucho tiempo. Utilizando lo que aprendiste hasta ahora acerca de **GNU parallel** buscas un modo de como convertir a **Ogg** todos los archivos **MP3** que están en la carpeta `/música`. Para esto necesitas procesar la lista de archivos **MP3**, remover la extensión `.mp3` y adicionar la nueva extensión `.ogg`. Después de varias simulaciones llegas a esta solución:

    ls /música/*.mp3 | parallel --dry-run ffmpeg -i {} {.}.ogg

Vamos a analizar la solución paso a paso.

Primero listas todos los archivos **MP3** en la carpeta `/música` con el comandos `ls /música/*.mp3` y luego redireccionas esta lista a la Entrada estándar usando el comando **pipe** `|`. Usas los símbolos `{}` y `{.}.ogg` para indicar el nombre original del archivo y para indicar el nuevo nombre del archivo respectivamente. Y finalmente activas la opción `--dry-run` todo el tiempo que estuviste experimentando, de esta forma logras simular lo que se ejecutará. Para finalmente convertir toda tu colección de música solo tienes que dejar de usar la opción `--dry-run`.

# Conclusión

**GNU** es una poderosa herramienta que está disponible para ayudar a ejecutar en paralelo comandos simples. No es necesario crear código de más de una linea para poder aprovechar el multiprocesamiento que tienen los dispositivos que usamos a diario. Si tienes quieres explorar aún más todas las opciones puedes dar un vistazo al manual `man parallel` y también a la [guía oficial][guía].

[sitio]: https://www.gnu.org/software/parallel/
[guía]: https://www.gnu.org/software/parallel/parallel_tutorial.html
[ffmpeg]: http://ffmpeg.org/
[^1]: Estos son nombres en clave, tu proyecto es de alto nivel de confidencialidad, por eso decieron dar nombre a sus servicios en honor a los 5 principales monumentos de Tiahuanacu: Puerta del Sol, Puerta de la Luna, Monolito Ponce, Monolito Fraile y Monolito Bennett.
[^2]: La inicialización demora este tiempo porque después de iniciar el servicio en un proceso separado *(fork)* se verifica si el servicio ya está listo para recibir peticiones, para esta verificación un `ping` satisfactorio es suficiente.
[^3]: Palabra en Aymara que traducida al español significa ¡Viva!.
[^4]: Ogg es un formato libre y abierto de medios, eso significa que no está sujeto a patentes de software como MP3.
