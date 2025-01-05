---
title: "ShareReplay en RxJS: ¡Comparte y Recuerda Emisiones en Angular!"
description: "¡Descubre el poder de shareReplay en RxJS!🔄️."
date: "Nov 05 2024"
---

# ¿Qué es? 📚

¿Alguna vez te has preguntado si hay una manera de reutilizar los datos de una petición sin tener que hacerla una y otra vez en Angular? Bueno, Hoy hablaremos de un operador RxJS que se llama shareReplay, que parece un truco de magia, pero en realidad es pura inteligencia RxJS. Así que ponte cómodo, vamos a aprender a usar `shareReplay`!

Para empezar, vamos a simplificar: `shareReplay` es un operador de RxJS que comparte y "recuerda" la última emisión de un observable. Imagina que shareReplay es como una grabadora que mantiene el último dato transmitido, y cada vez que alguien (otro suscriptor) lo pide, en vez de repetir todo desde el principio, simplemente lo entrega. ¡Ahorra tiempo y recursos! ⌛

## ¿Cómo usar `shareReplay` en Angular? 🛠️

Usar shareReplay en Angular es muy sencillo. Se puede usar cuando trabajamos con servicios que realizan peticiones HTTP. En aplicaciones Angular, los servicios son singleton por defecto, lo que significa que solo se crea una instancia durante la vida útil de la aplicación. Esto ya es bueno, pero `shareReplay` lleva esto al siguiente nivel.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, shareReplay } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class UserService {
  private users$!: Observable<User[]>;

  constructor(private http: HttpClient) {}

  public getUsers(): Observable<User[]> {
    if (!this.users$) {
      this.users$ = this.http.get<User[]>('https://api.example.com/users').pipe(
        shareReplay(1) // Aquí es donde la magia sucede 🪄
      );
    }
    return this.users$;
  }
}
```

> `shareReplay(1)` está ahí, como un guardián silencioso, asegurándose de que la petición HTTP solo ocurra una vez. Este número (1) es la cantidad de emisiones que queremos que "recuerde" el operador. En este caso, 1 es perfecto, porque solo necesitamos la última emisión.

## ¿Por qué usar `shareReplay` ? 🤔

Pongamos un caso realista: imagina que tienes una lista de usuarios que obtienes de un servidor. Sin `shareReplay`, cada vez que un componente necesita los usuarios, el servicio va al servidor y vuelve a hacer la misma petición. Esto es como pedir café cada vez que tomas un sorbo; ¡ineficiente y costoso! ☕

Con `shareReplay`, si alguien más necesita la misma lista, simplemente se entrega el resultado almacenado. No se hacen más peticiones al servidor. <mark>Esto es súper útil para datos que no cambian frecuentemente o que quieres cachear por un rato.</mark>

## ¿Cuándo no usar `shareReplay`? ⚠️

Es importante saber cuándo **NO** usar `shareReplay`, porque no siempre es el héroe que necesitamos. Aquí van algunos consejos:

* **Datos dinámicos**: Si los datos cambian constantemente (como los precios en una tienda), `shareReplay` podría hacer que otros usuarios vean datos viejos.
    
* **Datos sensibles**: Si estás trabajando con información que puede actualizarse en tiempo real, mejor usa operadores que te garanticen que recibes las actualizaciones más recientes ([switchMap](https://rxjs.dev/api/operators/switchMap), por ejemplo).
    

## Argumentos de `shareReplay` 💡

`shareReplay` no es solo "repetir y compartir", sino que también nos permite ajustar cómo se comporta al compartir datos entre suscriptores. Este operador tiene tres argumentos clave, y cada uno le da una personalidad distinta. ¿Listo para conocerlos? ¡Vamos a ello!

| **Nombre** | **Descripción** | **Tipo de dato** |
| --- | --- | --- |
| `bufferSize` | Número de emisiones que quieres recordar. | `number` |
| `windowTime` | Duración del tiempo que quieres recordar las emisiones. | `number` |
| `refCount` | controla el comportamiento de suscripciones. | [SchedulerLike](https://rxjs.dev/api/index/interface/SchedulerLike) |

### **bufferSize 💡**

El primer argumento de shareReplay es `bufferSize`, que define cuántas emisiones queremos recordar. Es útil cuando solo necesitas que el observable "recuerde" unas pocas emisiones recientes en lugar de todas. Piensa en esto como un historial de datos limitado: si estableces bufferSize en 1, solo recordará la última emisión; si lo configuras en 3, mantendrá las tres últimas, y así sucesivamente.

```typescript
import { of } from 'rxjs';
import { shareReplay } from 'rxjs/operators';

const source$ = of(1, 2, 3, 4).pipe(
  shareReplay(2) // Solo recuerda las últimas 2 emisiones
);

source$.subscribe(value => console.log('Suscriptor 1:', value));
source$.subscribe(value => console.log('Suscriptor 2:', value));

// Resultado en consola:
// Suscriptor 1: 3
// Suscriptor 1: 4
// Suscriptor 2: 3
// Suscriptor 2: 4
```

### windowTime 💡

El argumento `windowTime` te permite definir el tiempo durante el cual quieres que `shareReplay` recuerde las emisiones. En lugar de recordar un número fijo de valores, `windowTime` asegura que solo se recuerden las emisiones hechas dentro de un rango de tiempo específico.

Este parámetro es ideal para situaciones en las que quieres compartir emisiones recientes, pero solo por un tiempo limitado. Por ejemplo, si tienes datos de actualización constante, puedes hacer que los suscriptores reciban solo las últimas emisiones dentro de un lapso de tiempo, sin sobrecargar el sistema.

```typescript
import { interval } from 'rxjs';
import { take, shareReplay } from 'rxjs/operators';

const source$ = interval(1000).pipe(
  take(5),
  shareReplay({ windowTime: 2000 }) // Recuerda las emisiones de los últimos 2 segundos
);

setTimeout(() => {
  source$.subscribe(value => console.log('Suscriptor después de 3 segundos:', value));
}, 3000);

// Resultado en consola:
// Suscriptor después de 3 segundos: 3
// Suscriptor después de 3 segundos: 4
```

### refCount 💡

El tercer argumento, `refCount`, define el comportamiento de `shareReplay` respecto a la cantidad de suscriptores. Por defecto, cuando no está activado, `shareReplay` mantiene su suscripción a la fuente, incluso si no hay suscriptores, guardando el último valor emitido para futuras suscripciones.

Si `refCount` está en true, `shareReplay` se desuscribe automáticamente de la fuente cuando no hay suscriptores. Esto libera recursos, ya que evita que el observable siga activo sin nadie que lo escuche. Úsalo en casos donde no quieras mantener una conexión abierta si nadie está suscrito, por ejemplo, en conexiones de `WebSocket` o recursos de datos limitados.

```typescript
import { interval } from 'rxjs';
import { take, shareReplay } from 'rxjs/operators';

const source$ = interval(1000).pipe(
  take(5),
  shareReplay({ bufferSize: 1, refCount: true }) // Se desuscribe automáticamente si no hay suscriptores
);

const subscription = source$.subscribe(value => console.log('Suscriptor 1:', value));

setTimeout(() => {
  subscription.unsubscribe();
  console.log('Suscriptor 1 se desuscribió');

  setTimeout(() => {
    source$.subscribe(value => console.log('Suscriptor 2:', value));
  }, 2000);
}, 3000);

// Resultado en consola:
// Suscriptor 1: 0
// Suscriptor 1: 1
// Suscriptor 1: 2
// Suscriptor 1 se desuscribió
// Suscriptor 2: 0
```

## Conclusión 🎉

El operador shareReplay de RxJS es una herramienta fantástica en Angular para optimizar la carga de datos cuando puedes aprovechar el "replay" sin volver a hacer la petición. Ahorrarás peticiones, tus componentes cargarán más rápido, y la experiencia de usuario de tus aplicaciones van a mejorar mucho.

Si ya conocías este operador o es la primera vez, ¡Déjamelo saber en los comentarios!

## Referencias 🧾

[https://rxjs.dev/api/operators/shareReplay](https://rxjs.dev/api/operators/shareReplay)

[https://www.youtube.com/watch?v=mVKAzhlqTx8](https://www.youtube.com/watch?v=mVKAzhlqTx8)