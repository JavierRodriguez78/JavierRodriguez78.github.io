---
title:       "Php es Molón?"
subtitle:    "Php no está muerto sigue evolucionando."
description: "Regreso al blog después de más de 1 año sin publicar nada."
date:        2020-12-09
author:      "Xavi Rodríguez."
image:       "img/php.jpg"
tags:        ["PHP","laravel", "Symfony"]
categories:  ["Tech", "Opinión" ]
---
Aquí va otro post de **opinión**, pero desde la perspectiva de la experiencia ya que he trabajado durante los últimos proyectos en PHP, en algunos casos con [Laravel](https://laravel.com/) y en otros con [Symfony.](https://symfony.com/)
Tampoco quiero centrarme tanto en los frameworks sino más bién en el lenguaje y el ecosistema que lo envuelve que es bastante interesante.
<center>

![image-php](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSNDA-zXkIHWtfQIfvRf6sFGw6cvLtHnOMo1wre_G0UoKZmWJuZ)
</center>

## Un poco de historia.

[Php](https://es.wikipedia.org/wiki/PHP) ya es un lenguaje que lleva varias décadas entre nosotros, la primera vez que apareció fue en el año 1995. Pero desde mi punto de vista hay varios factores que han conseguido que este lenguaje interpretado sea bastante interesante.

El primero punto interesante, que ya conocéis todos es la adopción de la POO que el lenguaje a llevado a cabo desde ya hace unos años ya que las versiones anteriores a la 5 no implementaban este paradigma, así como la implementación de ciertas características que hacen que se acerque  a otros tipos de lenguajes de alto nivel como puede ser Java, C# etc.

En el momento de escribir este post la versión más recientes es la 7.4 donde el lenguaje ya tiene bastante madurez e incorpora aspectos tales como la posibilidad de declarar [arrow functions,](https://wiki.php.net/rfc/arrow_functions)
o que las propiedades de un objeto puedan ser tipadas, disponer de un operador [array spread](https://wiki.php.net/rfc/spread_operator_for_array), y un largo etc.

Pero hay otros aspectos que hacen que el lenguaje en si sea bastante interesante y es el ecosistema que hay alrededor de el, donde lo más destacable es el [PHP-FIG](https://www.php-fig.org/)

Esta es una asociación de diversos fabricantes de software así como de mantenedores de lenguaje que se creo con la intención de establecer un estándar en el desarrollo de software, y en su momento estaba conformada por grandes compañías como [Zend Software](https://www.zend.com/), Symfony, Laravel, [Magento](https://magento.com/), aunque en estos dos últimos años, parece que ha habido algún tipo de desacuerdo que ha producido la salida de Symfony y Laravel de dicho [grupo](https://www.php-fig.org/personnel/).

Sus especificaciones  o [PSR's](https://www.php-fig.org/psr/) siguen vigentes en el mercado de las cuales yo destacaría sobre todo dos que han hecho que el lenguaje y su ecosistema haya avanzado de forma muy considerable:

 - PSR-04:  Nos permite la posibilidad de crear  y establecer namespaces, y poder realizar autoload, lo cual abre muchas posibilidades al lenguaje como a los proyectos y la interoperatividad, hoy en día en un proyecto php puedes portar por ejemplo un componente de Symfony como puede ser [Forms](https://github.com/symfony/form) e incorporarlo siguiendo las reglas que establece dicho PSR, sin necesidad de instalar a priori Symfony
 
 
 - PSR-11: Este psr en particular ha abierto muchas puertas dentro de los frameworks actuales ya que define una interfaz para crear y gestionar  contenedores de dependencias, de esta manera poder realizar inyección de las mismas y abrir la posibilidad por ejemplo de realizar composición, o permitirnos desacoplar las diversas capas de nuestra aplicación para poder testear. Este punto nos podría dar otro post por entero.

Disponemos también de una herramienta que nos facilita la vida [composer](https://getcomposer.org/) que además de ser un gestor de dependencias también nos ayuda a mantener nuestro autoload de librerías en orden.

Y por último la cantidad de componentes y librerías disponibles que podemos comprobar en [packagist](https://packagist.org/)

Entre las que destacaría.

 - [**Php-di**](http://php-di.org/) Una librería que nos permite crear y gestionar un contenedor de dependencias.
 - [**Doctrine**](https://www.doctrine-project.org/) El [ORM](https://es.wikipedia.org/wiki/Mapeo_objeto-relacional) de Symfony.
 - [**Fast-Route**](https://github.com/nikic/FastRoute) Un gestor de rutas para php bastante completo.
 - etc.
 
 Todas estas herramientas nos permiten generar nuestro proyecto sin mayor complejidad y evitando el **php clásico** de código espagueti donde no existía ni siquiera división por capas.

Tampoco nos podríamos olvidar la cantidad de aplicaciones super conocidas basadas en este lenguaje:

 - [Wordpress.](https://es.wordpress.com/create/?v=spain_go_to_market&currency=EUR&utm_source=google&utm_campaign=google_wpcom_search_brand_desktop_es_es&utm_medium=paid_search&keyword=wordpress&creative=339382685172&campaignid=662707367&adgroupid=54953985106&matchtype=e&device=c&network=g&targetid=kwd-313411415&gclid=Cj0KCQjw9ZzzBRCKARIsANwXaeK89ywxWkUiatZZGEGx9ZwkfYAkIBx2tIXD1GAqmp-jZ43S7_H3znEaAspiEALw_wcB&gclsrc=aw.ds)
 - [Drupal.](https://www.drupal.org/)
 - [Magento.](https://magento.com/)
 - [SugarCrm.](https://www.sugarcrm.com/es/)
 - etc.

A la vez dispone de una gran diversidad de frameworks  donde podemos elegir uno u otro en función de nuestras necesidades.

 - Symfony
 - [Laminas.](https://getlaminas.org/)
 - Laravel.
 - [Yii.](https://www.yiiframework.com/)
 - [CakePhp](https://cakephp.org/).
 - [Phalcon.](https://phalcon.io/en-us)
 - etc.

## El futuro de php

La verdad es que la versión actual tendrá soporte como mínimo durante 2 años más hasta el [2022](https://www.php.net/supported-versions.php) , pero ya hace unos meses han anunciado el inicio de desarrollo de [php 8](https://wiki.php.net/rfc/php8), donde en algunos lugares ya han [dejado entrever](https://stitcher.io/blog/new-in-php-8) las nuevas features que aportaran al lenguaje.

Y yo destacaría sobre todo disponer de JIT(Just in time) el cual permitiría compilar partes de código en tiempo real, ellos dicen:

> Think of it like a "cached version" of the interpreted code, generated at runtime.

Será interesante ver como se materializa, pero de ser así dará un salto cualitativo al lenguaje.

## Conclusión.
Por todo lo visto anteriormente y muchas cosas más, para mí **Php** es un lenguaje molón, que todavía tiene mucho "dar que hablar",  y  aunque ya es mayor de edad, sigue siendo  una seria alternativa real y actual a otros lenguajes, o stacks más modernos, en el entorno **web**

Nos vemos en un próximo post!!!!

