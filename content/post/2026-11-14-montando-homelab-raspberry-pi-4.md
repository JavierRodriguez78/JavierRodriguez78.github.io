---
title: "Montando mi propio Homelab con Raspberry Pi 4: Docker, Pi-hole, Tailscale y Cloudflare"
description: "De una Raspberry Pi a una pequeña infraestructura doméstica: seguridad, monitorización, DNS y acceso remoto."
date: 2026-11-14
author: "Xavi Rodríguez"
tags: ["homelab", "raspberry-pi", "docker", "devops", "tailscale", "cloudflare", "pihole", "linux"]
categories: ["DevOps", "Homelab"]
cover:
  image: "img/homelab/homelab-raspberry-pi-architecture.png"
  alt: "Arquitectura del homelab con Raspberry Pi 4: servicios Docker, Tailscale, Cloudflare y monitorización"
  relative: false
draft: false
---
Hacía tiempo que tenía ganas de montar una infraestructura doméstica con la que poder experimentar, aprender y disponer de servicios propios sin depender constantemente de soluciones externas.

La idea inicial era sencilla: aprovechar una Raspberry Pi para centralizar servicios de red, mejorar la seguridad de mi conexión doméstica, monitorizar los dispositivos y disponer de acceso remoto cuando estuviera fuera de casa.

Lo que empezó como una instalación de Pi-hole ha terminado convirtiéndose en un pequeño Homelab basado en Docker, Debian, Tailscale y Cloudflare.

En este artículo voy a documentar la arquitectura, las decisiones técnicas y los principales componentes de la infraestructura.

![Raspberry Pi 4 utilizada como servidor doméstico](/img/homelab/raspberry-pi-4.jpg)

*Raspberry Pi 4 con 4 GB de RAM: el punto de partida de mi Homelab.*

## 1. Hardware y sistema operativo

El equipo principal es una Raspberry Pi 4 con 4 GB de RAM.

No es una máquina especialmente potente, pero resulta suficiente para ejecutar varios servicios ligeros en contenedores Docker.

| Componente | Configuración |
|---|---|
| Hardware | Raspberry Pi 4 |
| Memoria | 4 GB RAM |
| Sistema operativo | Debian 13 (Trixie) |
| Arquitectura | ARM64 |
| Contenedores | Docker Engine |
| Orquestación | Docker Compose |
| Red | Ethernet |
| Dirección IP | Estática |

He optado por Debian 13 para mantener el sistema ligero y reservar los recursos para los servicios.

## 2. Arquitectura general

La idea es separar las responsabilidades:

- La Raspberry ejecuta los servicios.
- Docker proporciona aislamiento y facilita el mantenimiento.
- Pi-hole gestiona el DNS y el bloqueo de publicidad.
- Homepage centraliza el acceso a los servicios.
- Uptime Kuma comprueba su disponibilidad.
- Glances proporciona información sobre los recursos del sistema.
- Cloudflare Tunnel permite establecer conectividad remota sin abrir puertos.
- Tailscale proporciona una VPN privada y permite utilizar la Raspberry como salida a Internet.

La arquitectura queda representada en el siguiente diagrama.

![Diagrama de arquitectura del Homelab con Raspberry Pi 4](/img/homelab/homelab-raspberry-pi-architecture.png)

*Arquitectura lógica del Homelab: servicios Docker, conectividad, acceso remoto y monitorización.*

Tailscale se ejecuta directamente en Debian, mientras que los servicios de aplicación están contenerizados.

Esta separación es intencionada: la conectividad de red de la propia Raspberry no debe depender de que un contenedor VPN esté funcionando.

## 3. Docker como base de la infraestructura

He elegido Docker Compose para gestionar los servicios.

Cada servicio tiene su propio directorio, con su configuración y su fichero `compose.yaml`.

```text
~/docker/
├── pihole/
│   └── compose.yaml
├── homepage/
│   ├── compose.yaml
│   └── config/
├── uptime-kuma/
│   └── compose.yaml
├── glances/
│   └── compose.yaml
├── dockerproxy/
│   └── compose.yaml
└── cloudflared/
    ├── compose.yaml
    └── secrets/
```

Esta organización facilita actualizar, detener, reconstruir y respaldar cada servicio de forma independiente.

## 4. Pi-hole: control DNS y bloqueo de publicidad

El primer servicio que instalé fue Pi-hole.

Pi-hole funciona como servidor DNS de la red doméstica y permite bloquear dominios asociados a publicidad, rastreadores y otros contenidos no deseados.

Su funcionamiento es relativamente sencillo:

1. Un dispositivo solicita resolver un dominio.
2. La petición llega a Pi-hole.
3. Pi-hole comprueba sus listas de bloqueo.
4. Si el dominio está bloqueado, devuelve una respuesta que impide su resolución.
5. Si está permitido, consulta el DNS ascendente.

![Panel de administración de Pi-hole](/img/homelab/pihole-dashboard.png)

*Pi-hole permite consultar las peticiones DNS, los dominios bloqueados y la actividad de los dispositivos.*

La interfaz de administración está disponible en:

```text
http://IP_DE_LA_RASPBERRY:8091/admin/
```

El DNS utiliza el puerto 53 TCP/UDP.

Una de las ventajas de tener Pi-hole centralizado es que no es necesario configurar un bloqueador individual en cada dispositivo.

Configurando el DNS de la red doméstica para utilizar Pi-hole, todos los dispositivos compatibles pueden beneficiarse del filtrado.

## 5. Homepage: un panel centralizado

Con varios servicios funcionando, necesitaba un punto de entrada desde el que acceder a todos ellos.

Para eso he utilizado Homepage.

Homepage es un dashboard autohospedado que permite organizar servicios, accesos directos e información de monitorización.

En mi caso, la idea es separar visualmente dos grandes áreas:

- **Infraestructura:** servicios que forman parte del propio Homelab.
- **Proyectos:** aplicaciones, webs y otros desarrollos personales que no pertenecen directamente a la infraestructura.

![Dashboard de Homepage](/img/homelab/homepage-dashboard.png)

*Homepage centraliza los accesos a los servicios y proyectos.*

Está disponible en:

```text
http://IP_DE_LA_RASPBERRY:3000
```

Además, Homepage puede integrar widgets de otros servicios, como Glances y Uptime Kuma.

## 6. Uptime Kuma: monitorización de disponibilidad

El siguiente componente es Uptime Kuma.

Su función es comprobar periódicamente que los servicios siguen disponibles.

Permite monitorizar diferentes protocolos y servicios:

- HTTP y HTTPS.
- TCP.
- DNS.
- Ping.
- Otros servicios mediante integraciones específicas.

![Panel de monitorización de Uptime Kuma](/img/homelab/uptime-kuma.png)

*Uptime Kuma permite consultar el estado y el historial de disponibilidad de los servicios.*

En este Homelab lo utilizo para centralizar la supervisión de los servicios principales.

La interfaz está disponible en:

```text
http://IP_DE_LA_RASPBERRY:3001
```

Una de las mejoras que quiero incorporar es monitorizar también la disponibilidad de Tailscale y Cloudflare Tunnel, de manera que pueda detectar problemas de conectividad y no solamente caídas de las aplicaciones web.

## 7. Glances: monitorización del sistema

Uptime Kuma comprueba si los servicios responden, pero también necesito conocer el estado de la propia Raspberry.

Para ello utilizo Glances.

Glances permite consultar información como:

- Uso de CPU.
- Memoria RAM.
- Espacio en disco.
- Procesos.
- Carga del sistema.
- Actividad de red.
- Información de sensores cuando están disponibles.

![Dashboard de Glances](/img/homelab/glances-dashboard.png)

*Glances ofrece una visión general del estado y consumo de recursos de la Raspberry Pi.*

Su interfaz web está disponible en:

```text
http://IP_DE_LA_RASPBERRY:61208
```

La integración con Homepage permite consultar información del sistema desde el panel centralizado.

## 8. Docker Socket Proxy: limitar el acceso a Docker

Homepage puede obtener información de los contenedores Docker.

Sin embargo, montar directamente el socket de Docker en una aplicación tiene implicaciones de seguridad importantes.

El socket de Docker proporciona un nivel de control muy elevado sobre el motor de contenedores.

Por eso he incorporado un Docker Socket Proxy.

La idea es ofrecer a las aplicaciones únicamente las operaciones de consulta que necesitan, evitando exponer directamente el socket completo.

El proxy permanece dentro de la red interna de Docker y no publica su puerto de administración en la red doméstica.

## 9. Acceso remoto: el problema del CGNAT

Una de las partes más interesantes del proyecto ha sido conseguir acceder a los servicios desde fuera de casa.

Inicialmente pensé en utilizar el clásico port forwarding del router.

Sin embargo, al revisar la configuración de la conexión descubrí que la dirección WAN del router pertenecía al rango `100.64.0.0/10`.

Esto es una señal de que la conexión está detrás de CGNAT.

Con CGNAT, el router no dispone de una dirección IPv4 pública directamente asignada que permita recibir conexiones entrantes mediante el port forwarding tradicional.

Por tanto, necesitaba otra solución.

He terminado utilizando dos tecnologías complementarias:

- Cloudflare Tunnel.
- Tailscale.

### 9.1 Cloudflare Tunnel

Cloudflare Tunnel permite establecer una conexión saliente desde la infraestructura doméstica hacia la red de Cloudflare.

El componente `cloudflared` se ejecuta en un contenedor Docker.

La conexión se establece desde casa hacia Cloudflare, por lo que no necesito abrir puertos entrantes en el router.

En este proyecto he utilizado una ruta privada para acceder a la red doméstica.

La red configurada es:

```text
RED_DOMESTICA/24
```

Esto permite acceder a los recursos privados de la red mediante la infraestructura de Cloudflare Zero Trust y WARP.

### 9.2 Tailscale como VPN privada

Para disponer de una VPN que permita tanto acceder a los servicios domésticos como utilizar la conexión de casa para navegar, he incorporado Tailscale.

Tailscale se instala directamente en Debian, fuera de Docker.

Una vez autenticada la Raspberry, he configurado dos funciones:

1. Exit Node.
2. Subnet Router.

La primera permite enviar el tráfico general de Internet a través de la Raspberry.

La segunda permite acceder a la red doméstica `RED_DOMESTICA/24`.

La configuración básica para anunciar la Raspberry como Exit Node es:

```bash
sudo tailscale set --advertise-exit-node
```

Y para anunciar la red doméstica:

```bash
sudo tailscale set --advertise-routes=RED_DOMESTICA/24
```

Después es necesario autorizar las opciones correspondientes desde la consola de administración de Tailscale.

En Android he instalado Tailscale y seleccionado la Raspberry como Exit Node.

El resultado es que, cuando estoy fuera de casa:

- Puedo navegar utilizando la conexión de Internet doméstica.
- Puedo acceder a los servicios de la Raspberry.
- Puedo acceder a otros dispositivos de la red doméstica mediante la ruta anunciada.

Esta combinación ha sido especialmente interesante porque resuelve dos necesidades diferentes con un único nodo doméstico.

## 10. Pi-hole y DNS cuando estoy fuera de casa

Una de las mejoras que quiero consolidar es utilizar Pi-hole también cuando accedo desde fuera de casa mediante Tailscale.

La arquitectura permite que el DNS remoto pueda resolverse utilizando el servidor Pi-hole de la red doméstica, siempre que el cliente tenga configurado ese servidor DNS y la conectividad esté correctamente establecida.

La dirección DNS de referencia sería:

```text
IP_DE_LA_RASPBERRY
```

En la administración de Tailscale se puede configurar un nameserver global o una configuración DNS adecuada para los clientes.

Es importante comprobar desde el móvil que las consultas llegan realmente a Pi-hole. Tener conectividad VPN no garantiza por sí solo que el cliente esté utilizando el DNS deseado.

## 11. Seguridad: firewall y exposición de servicios

Una infraestructura doméstica también necesita medidas de seguridad.

En este proyecto he utilizado UFW como firewall del sistema y reglas específicas de Docker para controlar el tráfico reenviado.

La política general es:

- Denegar conexiones entrantes por defecto.
- Permitir SSH únicamente desde la red doméstica.
- Permitir DNS desde los segmentos autorizados.
- Limitar el acceso a las interfaces web a la red doméstica.
- Controlar el tráfico que llega a los contenedores.
- No publicar servicios internos innecesariamente.
- Evitar exponer el Docker Socket Proxy.

También he mantenido las reglas necesarias para que Tailscale pueda gestionar el tráfico de la VPN y el encaminamiento de la red doméstica.

Una consideración importante: Docker puede modificar las reglas de iptables y el tratamiento del tráfico puede diferir del tráfico que atraviesa directamente UFW.

Por eso es necesario revisar las reglas de Docker y UFW conjuntamente, en lugar de asumir que una regla de UFW controla automáticamente todos los puertos publicados por Docker.

No conviene desactivar el firewall para solucionar un problema de conectividad sin entender antes por dónde circulan los paquetes.

## 12. Copias de seguridad automatizadas

Otro aspecto importante es disponer de una forma de recuperar la infraestructura.

He preparado un script de backup que recopila la configuración y los datos persistentes de los servicios.

El sistema utiliza:

- Un script de backup.
- Un temporizador de systemd.
- Ejecución diaria a las 03:00.
- Validación del archivo generado.
- Retención de las últimas siete copias.

La idea es que, si una actualización rompe un servicio o tengo que reinstalar el sistema, pueda recuperar su configuración sin empezar desde cero.

No se trata únicamente de guardar los ficheros Compose. Los datos persistentes y las configuraciones también deben formar parte de la estrategia de recuperación.

Las copias de seguridad deberían mantenerse también fuera de la Raspberry para protegerse frente a un fallo físico del dispositivo o del almacenamiento.

## 13. ¿Qué tenemos actualmente?

| Servicio | Tecnología | Función |
|---|---|---|
| Pi-hole | Docker | DNS y bloqueo |
| Homepage | Docker | Dashboard |
| Uptime Kuma | Docker | Disponibilidad |
| Glances | Docker | Monitorización |
| Docker Socket Proxy | Docker | Acceso limitado a Docker |
| Cloudflared | Docker | Túnel privado |
| Tailscale | Debian | VPN, Exit Node y Subnet Router |
| UFW | Debian | Firewall |
| systemd | Debian | Automatización de backups |

## 14. Próximos pasos

El Homelab todavía tiene recorrido. Algunas de las mejoras que me gustaría incorporar son:

- Configurar notificaciones de Uptime Kuma, por ejemplo mediante Telegram.
- Terminar de integrar todos los servicios y estados en Homepage.
- Verificar y consolidar el uso remoto de Pi-hole como DNS mediante Tailscale.
- Añadir almacenamiento externo para copias de seguridad.
- Incorporar métricas históricas de rendimiento.
- Estudiar la posibilidad de distribuir servicios entre varias Raspberry Pi.
- Mejorar la automatización de actualizaciones y mantenimiento.

La idea no es instalar servicios por instalarlos, sino construir una infraestructura que sea útil, segura y fácil de mantener.

## Conclusión

Este proyecto me ha servido para experimentar con tecnologías que normalmente encontramos en infraestructuras profesionales, pero trasladadas a un entorno doméstico.

Docker, DNS, VPN, túneles privados, firewall, monitorización y automatización son piezas que también forman parte del día a día de cualquier plataforma tecnológica.

Una Raspberry Pi 4 con 4 GB de RAM no sustituye a un servidor empresarial, pero es una plataforma fantástica para aprender, probar arquitecturas y construir servicios propios.

Y lo mejor es que el proyecto puede crecer poco a poco, incorporando nuevos servicios sin tener que rediseñar toda la infraestructura.

**Seguiremos ampliando el Homelab.**

## Referencias

- [Raspberry Pi](https://www.raspberrypi.com/)
- [Debian](https://www.debian.org/)
- [Docker](https://www.docker.com/)
- [Pi-hole](https://pi-hole.net/)
- [Homepage](https://gethomepage.dev/)
- [Uptime Kuma](https://github.com/louislam/uptime-kuma)
- [Glances](https://github.com/nicolargo/glances)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [Tailscale](https://tailscale.com/kb/)
- [UFW](https://help.ubuntu.com/community/UFW)
- [Hugo](https://gohugo.io/)
