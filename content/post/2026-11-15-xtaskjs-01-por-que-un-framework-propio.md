---
title: "xTaskJS (I): por qué he escrito mi propio framework para Node.js"
description: "Primer artículo de la serie sobre xTaskJS: de dónde nace, qué problema quiere resolver y qué aspecto tiene una aplicación mínima."
date: 2026-11-15
author: "Xavi Rodríguez"
tags: ["xtaskjs", "typescript", "nodejs", "framework", "inyeccion-de-dependencias", "open-source"]
categories: ["Proyectos", "xTaskJS"]
series: ["xTaskJS"]
cover:
  image: "img/xtaskjs/xtaskjs-logo.png"
  alt: "Logo de xTaskJS"
  relative: false
draft: true
---
En 2021 publiqué en este blog un [esqueleto de aplicación con TypeScript, Inversify y TypeORM]({{< ref "2021-12-11-Skeletor-typescript-DDD.md" >}}) y terminaba el artículo diciendo que me gustaría actualizarlo y hacerlo crecer.

Cuatro años después, aquella idea ha acabado convertida en algo bastante más grande: **xTaskJS**, un framework para Node.js escrito en TypeScript, con su propio contenedor de inyección de dependencias, 21 paquetes publicados en npm, un CLI y casi una treintena de aplicaciones de ejemplo.

Con este artículo empiezo una serie en la que voy a contar cómo está hecho por dentro, qué decisiones he tomado y qué he aprendido por el camino.

## ¿Por qué otro framework?

     - Primero quería entender como funcionaba un framework de las características de nestjs, symfony o Spring Boot internamente.
     - Segundo después de probar diferentes frameworks en diferentes lenguajes quería tener lo mejor de cada uno concentrado en un propio framework que me permita crear aplicaciones con todo lo bueno que aporta cada uno
     - Y por último xTaskjs, es un juego de palabras Task por todas las tareas que se pueden crear y js por su base en nodejs .

La respuesta corta es que quería entender de verdad qué hay debajo de un `@Controller()`.

Llevo años trabajando con contenedores de dependencias, tanto en PHP como en Node.js, y siempre desde el lado del usuario: declaras una clase, le pones un decorador y "mágicamente" aparece instanciada donde la necesitas. Escribir el contenedor, el escáner de clases y el ciclo de vida de la aplicación es la mejor forma que conozco de que esa magia deje de serlo.

La respuesta larga tiene que ver con las cosas que echaba de menos:

- Un modelo de componentes cercano al de Spring: `@Service()`, `@Repository()`, `@Controller()`, `@AutoWired()`, `@Qualifier()`, `@PostConstruct()` y `@PreDestroy()`.
- Descubrimiento automático de clases, sin tener que registrar cada proveedor a mano en un módulo.
- Libertad para elegir el servidor HTTP: `node:http` sin nada más, Express o Fastify, cambiando una línea.
- Piezas de arquitectura como CQRS, event sourcing o value objects disponibles como paquetes oficiales, y no como algo que cada proyecto reinventa.

## Qué aspecto tiene

Esta es una aplicación completa. Un fichero de arranque:

```typescript
import { CreateApplication } from "@xtaskjs/core";

async function main() {
  await CreateApplication({
    adapter: "node-http",
    autoListen: true,
    server: { host: "127.0.0.1", port: 3000 },
  });
}

main().catch((err) => {
  console.error("Error starting the application:", err);
});
```

Y un controlador:

```typescript
import { Controller, Get, Logger } from "@xtaskjs/common";

@Controller("/")
export class HealthController {
  constructor(private logger: Logger) {}

  @Get("/health")
  check() {
    this.logger.info("Health check endpoint called");
    return { status: "ok", timestamp: new Date().toISOString() };
  }
}
```

No hay ningún sitio donde se registre `HealthController`, ni donde se diga de dónde sale `Logger`. El framework escanea el proyecto, encuentra las clases decoradas, construye el grafo de dependencias y monta las rutas.

Cómo lo hace exactamente es el tema del siguiente artículo.

## El ecosistema

xTaskJS no es un único repositorio, sino cuatro piezas:

| Proyecto | Qué es |
|---|---|
| [xtask](https://github.com/xtaskjs/xtask) | Monorepo con el runtime, los paquetes oficiales y los ejemplos |
| `@xtaskjs/cli` | CLI con `create`, `generate`, `add` y `cache` |
| `typescript-starter` | Plantilla que descarga `xtask create` |
| [xtaskjs.io](https://xtaskjs.io) | Web de documentación, construida con el propio framework |

Y dentro del monorepo, los paquetes se agrupan más o menos así:

| Área | Paquetes |
|---|---|
| Núcleo | `core`, `common`, `config` |
| HTTP | `express-http`, `fastify-http`, `validation`, `throttler`, `socket-io` |
| Datos | `typeorm`, `cache` |
| Arquitectura | `value-objects`, `cqrs`, `event-source` |
| Infraestructura | `queues`, `scheduler`, `mailer`, `security`, `internationalization` |
| Integraciones | `bots`, `mcp` |
| Calidad | `testing` |

Todo se gestiona con `pnpm` y `turbo`, y se publica bajo licencia MIT.

## Cómo ha ido creciendo

El primer commit es del 27 de diciembre de 2025. Desde entonces el orden en el que han ido apareciendo las piezas dice bastante de cómo se construye un framework:

- **Diciembre de 2025**: estructura del monorepo y primeras publicaciones.
- **Febrero de 2026**: controladores HTTP y decoradores.
- **Marzo de 2026**: el mes más intenso. Adaptadores de Express y Fastify, seguridad con JWT y JWE, mailer, validación, internacionalización, scheduler, caché, colas, value objects y CQRS.
- **Abril de 2026**: event sourcing, Socket.IO y throttling.
- **Mayo de 2026**: rendimiento del arranque (escaneo en paralelo con worker threads, resolución perezosa del contenedor y manifiesto precompilado para producción), logger configurable, migraciones, bots, CI y el paquete de validación.
- **Junio de 2026**: servidor MCP con decoradores.

Primero lo que hace falta para responder una petición, después todo lo que una aplicación real necesita alrededor y, solo cuando ya había suficiente código como para notarlo, el rendimiento.

<!-- TODO Xavi: aquí encajaría muy bien una anécdota: qué paquete te costó más, qué tuviste que reescribir, qué harías distinto. -->

## Qué viene en la serie

Todos los artículos irán apareciendo agrupados en la [serie xTaskJS](/series/xtaskjs/). Este es el plan, que seguramente cambie por el camino:

1. **Por qué un framework propio** (este artículo).
2. **El contenedor de dependencias**: decoradores, metadatos, estereotipos y ciclo de vida.
3. **El arranque**: kernel, escaneo de clases y cómo bajar el tiempo de inicio con worker threads, resolución perezosa y manifiesto precompilado.
4. **Un núcleo, tres servidores**: los adaptadores de `node:http`, Express y Fastify.
5. **Value objects, CQRS y event sourcing** como paquetes del framework.
6. **Colas, scheduler y caché**: memoria, RabbitMQ y Redis.
7. **Seguridad**: JWT, JWE, guards y throttling.
8. **Testing**: un runtime en memoria para sobrescribir dependencias.
9. **Bots y MCP**: Telegram, Slack, WhatsApp y servidores MCP con decoradores.
10. **Mantener 21 paquetes siendo una sola persona**: monorepo, versionado, publicación en npm y CI.

Si quieres ir probándolo mientras tanto, el código está en [GitHub](https://github.com/xtaskjs/xtask) y la documentación en [xtaskjs.io](https://xtaskjs.io).
