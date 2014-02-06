---
layout: post
permalink: blog/ejecutando-comandos-en-paralelo
title: Ejecutando comandos en paralelo
categories: [hacking, productividad, tecnología]
tags:
---
# Ejecutando comandos en paralelo

En estos días casi todos los dispositivos electrónicos que utilizamos (computadores, portátiles, celulares) vienen con capacidades de multiprocesamiento. Como tecnólogo me es útil saber una forma simple y fácil de utilizar estos múltiples procesadores. Sé que puedo utilizar lenguajes de programación para distribuir múltiples tareas entre varios los procesadores, pero eso requiere escribir código. Buscando una forma aún más “fácil” que programar, encontré una interesante herramienta: [**GNU parallel**][sitio].

Existe una [guía][guía] explicando la mayoría de las características de esta herramienta. Esa guía es bastante extenso pero lamentablemente no es muy amigable. Entonces decidí escribir esta pequeña guía con la intención de que ayude a entender las características de esta poderosa herramienta.

![GNU parallel](/images/ejecutando-comandos-en-paralelo/gnu-parallel.png "GNU parallel")

## ¿Cómo instalar?

###Ubuntu

    sudo apt-get install parallel

### OSX

    brew install parallel

Después de la instalación tendremos acceso a un ejecutable: `parallel`. Para verificar que está instalado satisfacoriamente podemos ejecutar `parallel --version`, este comando mostrará una respuesta parecida a esta:

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

## Uso simple

Bueno, vamos intentar ejecutar dos comandos en paralelo utilizando **GNU parallel**:

    parallel ::: foo bar

En este caso los comandos (`foo` y `bar`) son enviados como parámetros para `parallel`. **GNU parallel** ejecuta los dos comandos en paralelo y concatena la respuesta de ambos. Los `:::` son importantes para diferenciar entre las opciones  y los comandos a ejecutar. Todo lo que está despues de los `:::` son los comandos que serán ejecutados en paralelo.

Ahora, si bien es útil ejecutar diferentes comandos en paralelo, muchas veces queremos ejecutar un mismo comando en paralelo pero enviando diferentes parámetros a este comando. Entonces hacemos lo siguiente: 

    parallel comando ::: parametro1 parametro2 parametro3

Por ejemplo en mi ultimo proyecto teníamos un comando para iniciar servicios Rest antes de ejecutar las pruebas automatizadas, llamábamos al comando `iniciar_servicio.sh` y como parámetro enviábamos en nombre del servicio que queríamos iniciar. Todo este proceso demoraba de 10 a 20 segundos para cada servicio[^1] y el problema era que teníamos varios servicios que iniciar. En este caso decidimos usar **GNU parallel** para ganar un poco de tiempo, lo que hicimos fue más o menos lo siguiente.

    parallel ./iniciar_servicio.sh ::: servicioA servicioB servicioC servicioD servicioE
    ./ejecutar_pruebas_automatizadas.sh

Ejecutamos el comando `iniciar_servicio.sh` con diferentes parámetros, en este caso 5 diferentes parámetros, que son los nombres de los diferentes servicioes a ser iniciados. **GNU parallel** iniciaba cada uno de los servicios por separado en paralelo y cuando todos los servicios estaban listos, entonces se ejecutaban las pruebas automatizadas con otro comando.

Si iniciáramos en secuencia los 5 servicios demoraría entre de **50 a 100 segundos** para tener todos funcionando, iniciándolos en paralelo la demora es fue de **10 a 20 segundos**.

### Entrada mediante archivo

Los comandos que se pasa a **GNU parallel** para ser ejeutados en paralelo pueden estar almacenados en un archivo. Por ejemplo en el archivo `comandos.txt` podemos tener lo siguiente:

    foo
    bar

Y ahora ejecutamos **GNU parallel** de la siguiente manera:

    parallel -a comandos.txt

### Entrada mediante entrada estándar

**GNU parallel** tambien puede recibir los comandos por la entrada estándar (stdin). Usando archivo del anterior ejemplo, podemos redireccionar su contenido para la entrada estándar para luego procesarlos en paralelo, de la siguiente manera:

    cat comandos.txt | parallel

### Manipulación de cadenas de caracteres (strings)

Un caso común es enviar a **GNU parallel** como parámetros rutas de archivos, en general porque queremos procesarlos en paralelo, para eso tenemos algunas formas de tratar estos parámetros.

Para hacer referencia al parámetro que **GNU parallel** utilizamos las llaves `{}`.

Como dije, en general se pasan rutas de archivos como parámetros. Una transformación que es útil es la de remover la extensión del archivo, esto se logra usando un punto dentro de las llaves `{.}`. A continuación un ejemplo.

Supongamos que tenemos en la carpeta `/musica` varios archivos en formato **MP3** que queremos convertir para el formato **Ogg**[^2], para esta tarea usaremos la herramienta **ffmpeg**. Para convertir un archivo de un formato a otro solo tenemos que especificar el nombre del nuevo archivo con su nueva extensión y **ffmepg** hará el resto del trabajo. Por ejemplo si queremos convertir nuestra canción favorita de **MP3** a **Ogg** sería más o menos así:

    ffmpeg -i cancion_favorita.mp3 cancion_favorita.ogg

Ahora que sabemos como convertir un archivo, tenemos que ver un modo como convertir a **Ogg** todos los archivos **MP3** que están en la carpeta `/musica`. Aquí es dónde entra **GNU parallel**. Para esto necesitamos procesar la lista de archivos **MP3**, remover la extensión `.mp3` y adicionar la nueva extensión `.ogg`. Para lograr esto usaremos `{}` y `{.}` de la siguiente forma:

    ls *.mp3 | parallel ffmpeg -i {} {.}.ogg

## Conclusión

**GNU** es una poderosa herramienta que está disponible para ayudar a ejecutar en paralelo comandos simples. No es necesario crear código de más de una linea para poder aprovechar el multiprocesamiento que tienen dos dispositivos que usamos a diario. Si tienes quieres explorar aún más todas las opciones puedes dar un vistazo al manual `man parallel` y también a la [guía oficial][guía].

[sitio]: https://www.gnu.org/software/parallel/
[guía]: https://www.gnu.org/software/parallel/parallel_tutorial.html
[^1]: Como parte de la inicialización también se verificaba si el servicio ya estaba listo para recibir peticiones, para esto un `ping` satisfactorio era suficiente.
[^2]: Ogg es un formato libre y abierto de medios, eso significa que no está sujeto a patentes de software como MP3.
