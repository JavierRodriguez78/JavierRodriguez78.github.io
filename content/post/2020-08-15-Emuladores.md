---
title:       "Emuladores"
subtitle:    "Vamos a entender como funcionan los emuladores."
description: "Artículo sobre emuladores."
date:        2020-08-15
author:      "Xavi Rodríguez."
image:       "img/retro.jpg"
tags:        ["Emuladores","Zx", "chip-8", "cpc-464"]
categories:  ["Tech"]
---

Hace tiempo que quería meterme en el mundo de los emuladores y la verdad es que siempre me han llamado bastante la atención.  Por la complejidad técnica que conlleva tanto conocer el sistema que quieres emular, como por el desarrollo en sí.

Además ya que tengo cierta edad, siempre me cae alguna lagrimita al ver una fotografía del [CPC 464 de Amstrad](https://es.wikipedia.org/wiki/Amstrad_CPC_464). 
Con el que empecé mis pinitos con 12 años y me inició en el mundo de la programación  ya que con este equipo le acompañaba un  [libro](http://www.cpcmania.com/Docs/Manuals/Manual%20de%20Usuario%20Amstrad%20CPC%20464.pdf) de [Basic](https://es.wikipedia.org/wiki/BASIC).
Al final del libro siempre había juegos escritos para que lo portaras a tu equipo.
  
<center>![cpc](https://esacademic.com/pictures/eswiki/51/300px-Amstrad_CPC464.jpg)</center>


**Antes de nada indicar que no soy especialista de ello**, simplemente en éste post recojo información sobre emulación para poder entender los conceptos básicos, así que voy a ello.

## ¿Qué es un emulador?
Si nos vamos a la definición de la Wikipedia:

>En [informática](https://es.wikipedia.org/wiki/Inform%C3%A1tica "Informática"), un **emulador** es un _[software](https://es.wikipedia.org/wiki/Software "Software")_ que permite ejecutar [programas](https://es.wikipedia.org/wiki/Programa_inform%C3%A1tico) o [videojuegos](https://es.wikipedia.org/wiki/Videojuego "Videojuego") en una plataforma (sea una arquitectura de _[hardware](https://es.wikipedia.org/wiki/Hardware "Hardware")_ o un [sistema operativo](https://es.wikipedia.org/wiki/Sistema_operativo "Sistema operativo")) diferente de aquella para la cual fueron escritos originalmente. A diferencia de un [simulador](https://es.wikipedia.org/wiki/Simulador "Simulador"), que solo trata de reproducir el comportamiento del programa, un emulador trata de modelar de forma precisa el dispositivo de manera que este funcione como si estuviese siendo usado en el aparato original.

Lo primero que hay que tener en cuenta que un **Emulador** no es un **Simulador** y por tanto debemos intentar programar el funcionamiento interno de la forma más fidedigna  de aquello que queremos emular, comportamiento de la **cpu** llamadas a memoria, etc.

## ¿Cómo funciona un emulador?
Para entender como funciona un **Emulador** debemos saber que este se encargará de traducir las instrucciones para que un sistema para el que no fue pensado originalmente, pueda  entender y ejecutar  esa lista de instrucciones que son los programas o juegos a emular.

Para ello **Emularará** el sistema original  ejecutando las  instrucciones **[opCodes](https://es.wikipedia.org/wiki/C%C3%B3digo_de_operaci%C3%B3n)** de la **cpu**


<center> 
![opCodes](http://i.imgur.com/VUyLG31.png)
</center>


## ¿Tipos de emulación?
Podemos encontrarnos con diferentes clasificaciones de emuladores :

 - Emulación a bajo nivel o **[LLE](http://emulation.gametechwiki.com/index.php/High/Low_level_emulation)** se define cuando emulamos todos los componentes del sistema, cpu, memoría, etc.
 - Emulación a alto nivel o **[HLE](http://emulation.gametechwiki.com/index.php/High/Low_level_emulation)** se podría definir emulación de alto como una simulación, los volcados de BIOS y otros códigos específicos de la máquina sin entrar a emular al detalle todos los procesos del sistema.

Entre las ventajas de la emulación  **HLE**, principalmente está la capacidad de utilizar el hardware del host mucho mejor y más fácilmente, la capacidad de optimizar los resultados a medida que mejoran el código y el hardware.

 EL progreso de las implementaciones también es mucho más independiente de la documentación detallada del hardware, en lugar de depender solo de la lista de posibles funciones disponibles para el programador.

Las desventajas incluyen una dependencia mucho mayor en la estandarización entre las aplicaciones de destino y la presencia de mecanismos suficientemente de alto nivel en las plataformas emuladas. Si no existe tal mecanismo, o si las aplicaciones no lo utilizan de una de las formas ya admitidas, no funcionarán correctamente.

## ¿Por donde empezar?
En un principio quería lanzarme a lo grande y empezar por la emulación de un **[Spectrum ZX](https://es.wikipedia.org/wiki/Sinclair_ZX_Spectrum)** para los viejos del lugar sabrán que fue un equipo que nos quito muchas horas de sueño, por los juegos y por los procesos de carga. XDDD

Pero revisando la documentación, y sin tener idea sobre emulación, etc, decidir bajar a la tierra y empezar por algo más sencillo para posteriormente escalar. Y que además nos ayude a entender todo el proceso y el funcionamiento de la emulación.
Así que he decidido empezar por el **[CHIP-8](https://es.wikipedia.org/wiki/CHIP-8)** que contiene un juego de opCodes  que me permitirá jugar y conocer todo el proceso de funcionamiento interno.

## ¿Cómo lo voy hacer?

Pues utilizando un lenguaje que cada vez me sorprende más [Go](https://golang.org/).
Así que en un próximo post hablaremos más en detalle del **CHIP-8** y empezaremos a estructurar el proyecto. 

Hasta pronto.