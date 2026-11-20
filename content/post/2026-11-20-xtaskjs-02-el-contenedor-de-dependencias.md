---
title: "xTaskJS (II): el contenedor de dependencias"
description: "Segundo artículo de la serie sobre xTaskJS: cómo funciona por dentro el contenedor de inyección de dependencias, con decoradores, metadatos, estereotipos y ciclo de vida."
date: 2026-11-20
author: "Xavi Rodríguez"
tags: ["xtaskjs", "typescript", "nodejs", "framework", "inyeccion-de-dependencias", "decoradores", "open-source"]
categories: ["Proyectos", "xTaskJS"]
series: ["xTaskJS"]
cover:
  image: "img/xtaskjs/xtaskjs-logo.png"
  alt: "Logo de xTaskJS"
  relative: false
draft: false
---
En el [artículo anterior]({{< ref "2026-11-15-xtaskjs-01-por-que-un-framework-propio.md" >}}) te conté por qué me lancé a escribir mi propio framework. Decía que quería entender de verdad qué hay debajo de un `@Controller()`. Pues hoy toca abrir el capó.

Y empezamos por el corazón: **el contenedor de dependencias**. Casi todo lo demás en xTaskJS (controladores, colas, CQRS, bots...) se apoya en él. Lo bueno es que no es tan grande como parece: son unas 850 líneas, repartidas en pocos ficheros dentro de `packages/core/src/di`.

| Fichero | Qué hace |
|---|---|
| `component.ts` | El decorador base `@Component` y la lectura de sus metadatos |
| `stereotypes.ts` | `@Service`, `@Controller` y `@Repository` |
| `autowired.ts` | `@AutoWired`, para inyectar en propiedades |
| `qualifier.ts` | `@Qualifier`, para elegir una dependencia por nombre |
| `lifecycle.ts` | `@PostConstruct` y `@PreDestroy` |
| `container.ts` | El contenedor: registro, resolución y destrucción |

## Un decorador no hace magia, solo deja una nota

Lo primero que hay que quitarse de la cabeza es que `@Service()` "hace algo". No registra nada ni crea ninguna instancia. Lo único que hace es **dejar una nota pegada en la clase**, usando `reflect-metadata`:

```typescript
const COMPONENT_KEY = Symbol("xtaskjs:component");

export function Component(options: ComponentOptions = {}) {
  return function (target: any) {
    Reflect.defineMetadata(COMPONENT_KEY, options, target);
  };
}
```

Y ya está. La clase queda marcada con sus opciones (`scope`, `name`, `type`...) y, más adelante, el contenedor leerá esa nota y decidirá qué hacer con ella. Separar *marcar* de *actuar* es la idea más importante de todo el artículo: los decoradores solo declaran, el que manda es el contenedor.

Hay un segundo truco, y este lo pone el compilador. Con `emitDecoratorMetadata` activado en el `tsconfig`, TypeScript apunta él solito, en cada clase decorada, de qué tipo es cada parámetro del constructor. Gracias a eso el contenedor sabe, sin que se lo digas, que este servicio necesita un `ApiClient`:

```typescript
@Service()
class UserService {
  constructor(private readonly apiClient: ApiClient) {}
}
```

Si no activas `experimentalDecorators` y `emitDecoratorMetadata`, esto no funciona. Más adelante te cuento por qué eso me da un poco de vértigo.

## Estereotipos: Service, Controller y Repository

Los tres son azúcar sobre `@Component`. Lo único que añaden es una etiqueta `type`:

```typescript
export const Service = (options = {}) =>
  Component({ ...options, type: "service" });

export const Controller = (options = {}) =>
  Component({ ...options, type: "controller" });

export const Repository = (options = {}) =>
  Component({ ...options, type: "repository" });
```

Vengo de Spring y de Symfony, donde esta distinción sirve para contar la intención del código: un `@Repository` habla con los datos, un `@Service` lleva la lógica de negocio.

Eso sí, te lo digo tal cual: **hoy el contenedor trata los tres exactamente igual**. El `type` es una etiqueta para quien lea el código y para otros paquetes. Lo que sí hace especial a un controlador son los metadatos de rutas de `@xtaskjs/common`, y eso lo recoge el servidor HTTP, no el contenedor.

## Cómo se registra y cómo se construye una clase

Cuando el contenedor registra una clase **no la instancia**. Lo que guarda es un *proveedor*: una función que sabe cómo construirla cuando alguien la pida. Y cuando la piden, pasa esto:

1. Mira que la clase no esté ya construyéndose. Si lo está, hay una dependencia circular y lanza un error con la cadena completa, tipo `Circular dependency detected: A -> B -> A`.
2. Lee los tipos del constructor y resuelve cada dependencia.
3. Hace el `new` con todo lo que necesita.
4. Inyecta las propiedades marcadas con `@AutoWired`.
5. Ejecuta el `@PostConstruct`, si lo hay.
6. Si hay `@PreDestroy`, apunta la instancia para llamarlo cuando la aplicación se apague.

### Singleton o transient

| Scope | Qué pasa |
|---|---|
| `singleton` (el de por defecto) | Se crea una única instancia y se reutiliza siempre |
| `transient` | Se crea una instancia nueva cada vez que se pide |

```typescript
@Service({ scope: "transient" })
class RequestIdService {
  private readonly requestId = Math.random().toString(16).slice(2);

  value(): string {
    return this.requestId;
  }
}
```

### Dependencias perezosas

Esta es una decisión de la que estoy bastante contento. Por defecto (`resolutionStrategy: "lazy"`), el contenedor no construye las dependencias del constructor en ese momento. Te entrega un `Proxy` que **crea la dependencia de verdad la primera vez que la usas**.

Eso me da dos cosas:

- La aplicación arranca más rápido, porque no se construye todo el grafo de golpe. De esto hablaré en el artículo 3.
- Se evitan muchos problemas con el orden en que se construyen las clases.

Si prefieres el comportamiento de toda la vida, con `resolutionStrategy: "eager"` se resuelve todo al instante. Además, el contenedor mide cuánto tarda en crearse cada componente (`getInstantiationMetrics()`), que viene genial para cazar al que te ralentiza el arranque.

Una excepción a propósito: las dependencias con `@Qualifier` se resuelven siempre al momento, porque un proxy no es `===` a la instancia real y quiero que la referencia se conserve.

## Dos formas de inyectar

La primera es por constructor, que es la que yo recomiendo: las dependencias quedan a la vista y el objeto nace siempre completo.

```typescript
@Service()
class ReportService {
  constructor(@Qualifier("logger") private readonly logger: ILogger) {}
}
```

La segunda es por propiedad, con `@AutoWired`. Es cómoda cuando no quieres un constructor kilométrico:

```typescript
@Service()
class ReportService {
  @AutoWired()
  private readonly userService!: UserService;

  @AutoWired({ required: false })
  private readonly metrics?: MetricsService;
}
```

Con `required: false`, si la dependencia no existe, el campo se queda vacío y no pasa nada. Si es obligatoria (lo normal), el error te dice exactamente qué propiedad ha fallado.

Y `@Qualifier("nombre")` sirve para pedir un componente concreto por el nombre con el que se registró (`@Service({ name: "..." })`) o una instancia que hayas registrado a mano con `registerNamedInstance`. Ahora mismo `@Qualifier` se usa en parámetros del constructor; en propiedades se hace con `@AutoWired({ qualifier: "..." })`.

## El ciclo de vida

Tengo dos ganchos, con la misma idea que en Spring:

```typescript
@Service()
class DatabaseService {
  @PostConstruct()
  connect() {
    // Se ejecuta cuando la instancia ya está lista
  }

  @PreDestroy()
  disconnect() {
    // Se ejecuta cuando la aplicación se apaga
  }
}
```

`@PostConstruct` se ejecuta justo después de construir la clase y de inyectarle sus propiedades, así que ya puedes usar tus dependencias dentro. `@PreDestroy` se ejecuta al cerrar la aplicación, **en orden inverso al de creación**: lo último que se levantó es lo primero que se baja. Es justo lo que quieres cuando tienes conexiones que dependen unas de otras.

## Lo que todavía me falta

Al repasar el código para escribir este artículo me encontré cosas que prefiero contarte yo antes de que las descubras tú:

- **`primary` y `condition` existen en las opciones, pero el contenedor todavía no las usa al resolver.** Están declaradas en el tipo, y `primary` se anota internamente, pero no desempatan ni activan nada de forma condicional. Es trabajo pendiente, no una funcionalidad.
- **`@PreDestroy` en componentes `transient`:** cada instancia se apunta para destruirla al final, así que en una aplicación que viva mucho tiempo y cree muchas instancias transitorias, esa lista irá creciendo. Lo tengo que acotar.
- **`@PostConstruct` es síncrono.** Si tu inicialización es asíncrona, el contenedor no la espera.
- **Solo cabe un `@PostConstruct` y un `@PreDestroy` por clase.**
- **Los decoradores son los experimentales de TypeScript** y dependen de `reflect-metadata`. Los decoradores estándar de TC39 no emiten los tipos de las dependencias, así que migrar a ellos me obligará a replantear cómo descubre el contenedor lo que necesita cada clase. Ese es el vértigo del que hablaba antes.

Creo que contarlo así suma más que resta: prefiero enseñar lo que falta que fingir que está todo terminado.

## Lo que viene

Ya hemos visto cómo se *registra* y se *construye* un componente. Pero queda una pregunta importante: si tú nunca registras tus clases a mano, ¿cómo sabe el contenedor que existen?

En el siguiente artículo, **"El arranque"**, te cuento cómo escanea el proyecto, por qué lo hago con *worker threads* y qué trucos uso para reducir el tiempo de inicio.

Mientras tanto, el código está en [GitHub](https://github.com/xtaskjs/xtask) y la documentación en [xtaskjs.io](https://xtaskjs.io). Si lo pruebas y algo no cuadra con lo que cuento aquí, escríbeme: es la mejor forma de que mejore. Y recuerda que todos los artículos van apareciendo en la [serie xTaskJS](/series/xtaskjs/).
