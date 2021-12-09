---
title:       "Monitorizando nuestro servidores con Grafana, Influx Telegraph"
subtitle:    "2º Parte"
description: "Como podemos monitorizar nuestra aplicación web."
date:        2020-08-01
author:      "Xavi Rodríguez."
image:       "img/monitoring.jpg"
tags:        ["Nginx","Grafana", "Influx", "Telegraph"]
categories:  ["Tech"]
---


Bueno vamos a acceder ahora al servidor cliente para poder enviarle métricas de nuestro servidor para ello vamos a utiliza telegraph 
Configuramos nginx comprobamos si los módulos para geo y status de nginx estan activados.
Añadimos el ćodigo suficiente para poder comprobar el status de nginx, para ello nos dirigimos a la carpeta 
```bash
$> /etc/ningx/status
```
Creamos un archivo denominado status.conf
```
server { 
  listen 127.0.0.1:9090;  
  location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.0/24;  
  }
}
```
Comprobamos que funciona mediante el siguiente comando.
```bash
$> curl 127.0.0.1:9090/nginx_status
```
Nos devolverá información .

Para la geolocalización crearemos un directorio dentro de nginx denominado geoip, en la siguiente localización:
```bash
$> /etc/nginx/geoip 
```
Nos descargamos los siguientes archivos
 ```bash
$> wget https://web.archive.org/web/20181119091911/http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
$> wget https://web.archive.org/web/20181119091911/http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
```
Una vez descargados los descomprimimos con:
```bash
$> gunzip GeoIP.dat.gz
$> gunzip GeoLiteCity.dat.gz
```
Ahora vamos activar la geolocalización en nuestro nginx, para ello debemos modificar el nginx.conf
Modificamos los siguientes parámetros
```
        log_format vhosts '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$request_time" "$upstream_connect_time" "$geoip_city" "$geoip_city_country_code"';
        access_log /var/log/nginx/access.log vhosts;
        error_log /var/log/nginx/error.log;


        # Add GEO IP support 
        geoip_country /etc/nginx/geoip/GeoIP.dat; # the country IP data
        geoip_city /etc/nginx/geoip/GeoLiteCity.dat; # the city IP data

```

Y si en nuestros host debemos comentar los comentarios para ello nos dirigimos sites-enabled y comentamos las líneas de los logs
```

        #access_log /var/log/nginx/access.log;
        #error_log /var/log/nginx/error.log;

```

1º paso es instalar telegraph en nuestro server, vamos a utilizar los repositorios de influx data para ello ejecutaremos.
```bash
$> wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
$> echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

2º E instalamos telegraph
```bash
$> sudo apt-get update && sudo apt-get install telegraf
$> sudo service telegraf start 
```

3º Comprobamos que telegraph se encuentra levantado.
```bash
$> service telegraf status
``` 
Nos debería indicar que se encuentra en funcionamiento.

4º Vamos a configurar telegraph para que envíe las meicas de forma adecuada a nuestro grafana, para ello nos dirigimos al directorio
```bash
$>/etc/telegraph
```
Y editamos el archivo telegraf.conf añadiendo al final las siguientes líneas.
Debemos sustituir:

 - Urls la ip de nuestra máquina bastión.
 - username: el username que tengamos configurado.
 - password: el password que hayamos configurado.

Una vez realizado sólo tenemos que reiniciar el servicio mediante.
```bash
$>service telegraf restart
```
Y ya podemos comprobar en grafana en el dashboard previamente instalado si ya nos llegan los datos de esta máquina.
