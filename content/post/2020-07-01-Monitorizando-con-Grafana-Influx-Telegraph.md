---
title:       "Monitorizando nuestro servidores con Grafana, Influx Telegraph"
subtitle:    "1º Parte"
description: "Como podemos monitorizar nuestra aplicación web."
date:        2020-07-01
author:      "Xavi Rodríguez."
image:       "img/monitoring.jpg"
tags:        ["Nginx","Grafana", "Influx", "Telegraph"]
categories:  ["Tech"]
---

Bueno muchas veces tenemos que monitorizar diveras apps, y para no estar pasando de un pantalla a otra nos interesa tenerlo centralizado en un único punto.

En la máquina Bastión

1º  Creamos un directiorio dentro de:
```bash
$>\opt\monitoring
```
En el crearemos el siguiente script para docker-compose
```yaml
version: "2"  
services:  
  grafana:  
    image: grafana/grafana  
    container_name: grafana  
    restart: always  
    ports:  
      - 3000:3000  
    networks:  
      - monitoring  
    volumes:  
      - grafana-volume:/var/lib/grafana influxdb:  
    image: influxdb  
    container_name: influxdb  
    restart: always  
    ports:  
      - 8086:8086  
    networks:  
      - monitoring  
    volumes:  
      - influxdb-volume:/var/lib/influxdbnetworks:  
  monitoring:volumes:  
  grafana-volume:  
    external: true  
  influxdb-volume:  
    external: true
```
El cual nos crea dos contenedores uno con grafana y otro con influxdb.

Ahora debemos crear la red y los volúmenes donde vamos a compartir nuestros datos del contenedor y que se mantengan persistentes.
```bash
$> docker network create monitoring  
$> docker volume create grafana-volume  
$> docker volume create influxdb-volume
```
Podemos comprobar si se ha creado los volúmenes ejecutando:
```bash
$> docker volume ls
```
Y la creación del la red interna.
```bash
$> docker network ls
```
Ok en este momento vamos a inicializar los contenedores y los volúmenes para parsear los datos de acceso, una vez creados los contenedores automáticamente se eliminarán.
```bash
$> docker run --rm \  
-e INFLUXDB_DB=telegraf -e INFLUXDB_ADMIN_ENABLED=true \  
-e INFLUXDB_ADMIN_USER=admin \  
-e INFLUXDB_ADMIN_PASSWORD=supersecretpassword \  
-e INFLUXDB_USER=telegraf -e INFLUXDB_USER_PASSWORD=secretpassword \  
-v influxdb-volume:/var/lib/influxdb \  
influxdb /init-influxdb.sh
```

Y por último una ya podemos lanzar nuestro servicio con docker-compose.
```bash
$> docker-compose up
```
Podemos comprobar si está todo correcto si nos dirigimos a:
**http://[ip de nuestra máquina]:3000**
Y comprobamos que tenemos acceso a **Grafana.**
![https://miro.medium.com/max/1454/0*IB_2fT2-mQihHP0l](https://miro.medium.com/max/1454/0*IB_2fT2-mQihHP0l)
Los datos de acceso son

 - username:admin
 - password:admin

Nos pedirá que cambiemos los datos de acceso:
![https://kifarunix.com/wp-content/uploads/2019/01/grafana-reset-pass.png](https://kifarunix.com/wp-content/uploads/2019/01/grafana-reset-pass.png)

Ok una vez dentro lo primero que debemos configurar es nuestra fuente de datos que en nuestro aso será **influxdb**. Para ello seleccionaremos añadir una fuente de datos.
![https://miro.medium.com/max/1833/0*en5CBeaGTKe9Qvfu](https://miro.medium.com/max/1833/0*en5CBeaGTKe9Qvfu)
Seleccionamos **InfluxDB** y configuramos los siguientes campos:

 - URL: https://influxdb:8086
 - Database: telegraf
 - User: telegraf
 - Password: secretpassword
 ![https://miro.medium.com/max/915/0*FECckMUhT-v_vaXH](https://miro.medium.com/max/915/0*FECckMUhT-v_vaXH)
Cuando ya tengamos los datos, pulsamos sobre el botón save/test, si todo ha ido bién debería aparecer un mensaje como el siguiente:
![https://miro.medium.com/max/1103/0*t02qDBYSGrqP2dew](https://miro.medium.com/max/1103/0*t02qDBYSGrqP2dew)

Bueno vamos a instalar nuestro primer dashboard predefenido para obtener los datos de las máquinas.
Para ello vamos a descarganos un dashboard ya creado desde la propia web de grafana, para ello nos dirigiremos a:
[https://grafana.com/grafana/dashboards/928](https://grafana.com/grafana/dashboards/928)
E importaremos este dashboard a nuestro grafana, en la pantalla principal pulsamos importar e indicamos el id del dashboard
![https://miro.medium.com/max/374/0*5-AP0Xow6lfvvc0E](https://miro.medium.com/max/374/0*5-AP0Xow6lfvvc0E)

Una vez importado debemos seleccionar la fuente en nuestro caso influxdb.
![](https://miro.medium.com/max/1079/0*c1azaaywigf9RPjp)
