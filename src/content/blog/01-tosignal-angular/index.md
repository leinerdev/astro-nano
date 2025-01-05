---
title: "Aprende a Convertir Observables en Signals con toSignal en Angular."
description: "Aprende cómo el enfoque toSignal cambia el manejo de observables en Angular."
date: "Nov 04 2024"
---

¡Bienvenidos a la fiesta de **Angular Signals**! 🎉 Estas nuevas joyitas traen consigo un montón de mejoras que harán que la reactividad y la gestión del estado en nuestras aplicaciones Angular brillen como nunca antes. En este emocionante recorrido, vamos a descubrir algunas de las características más destacadas y cómo se integran perfectamente en el universo de Angular. ¡Prepárense para darle un toque mágico a su aprendizaje en programación! ✨

### 🤓 Repasemos que es un Observable

En el mundo de **Angular**, un observable es como un canal de comunicación que nos permite recibir datos de forma asíncrona. Imagina que es un grifo: puedes abrirlo y cerrar el flujo de información cuando lo necesites. Con un observable, puedes suscribirte a eventos, como cambios de datos o acciones del usuario, y reaccionar a ellos en tiempo real.

### 🤔 Signal, ¿Qué es?

En el ecosistema de Angular, un **Signal** es una herramienta poderosa que permite manejar el estado de manera reactiva y eficiente. Imagina que es como una linterna: en lugar de buscar en la oscuridad, puedes encenderla para ver exactamente lo que necesitas.

### 🤔 ¿Quién es ese tal `toSignal()` ?

Imagina que tienes un canal de noticias y quieres estar al tanto de la información más reciente sin tener que estar pegado a la pantalla todo el día. ¡Eso es exactamente lo que hace `toSignal` en Angular!

`toSignal` es un operador que transforma un Observable en un Signal, lo que te permite acceder a los valores producidos por ese Observable de manera reactiva y sincronizada. Esto significa que siempre tendrás el valor más reciente emitido, como si tuvieras un asistente personal que te informa de inmediato sobre cualquier novedad.

Si el Observable experimenta algún error, `toSignal` lo manejará y lanzará una alerta, asegurándose de que no te pierdas nada importante. En resumen, `toSignal` facilita la vida en Angular, permitiéndote trabajar con datos de manera más eficiente y con un código más limpio.

### 🚀 ¡Pongámoslo en práctica!

Vamos a crear un servicio sencillo para obtener un listado de usuarios, en este punto seguramente eres un crack usando servicios en **Angular 😉.**

```typescript
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'https://jsonplaceholder.typicode.com/users';

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
}
```

> Como puedes notar, el método `getUsers()` devuelve un observable con un listado de usuarios, bastante sencillo y conocido hoy en día.

En **Angular**, uno de los principios más útiles que seguimos es el **quinto principio SOLID**, que nos ayuda a inyectar servicios en nuestros componentes de manera efectiva. ¡Es realmente sensacional! 🌟

```typescript
@Component({
  selector: 'app-user-list',
  template: `
    <h2>Lista de Usuarios</h2>
    <ul>
      @for(user of users; track user) {
        <li>{{ user.name }}</li>
      }
    </ul>
  `
})
export class UserListComponent implements OnInit {
  users: User[] = [];

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.userService.getUsers().subscribe((data) => {
      this.users = data;
    });
  }
}
```

> En este punto consumimos una API de usuarios de manera convencional y comúnmente conocida en Angular.

¿No crees que tener tantas líneas de código solo para obtener una lista de usuarios de una API es un poco excesivo? 🤔 ¡Es como usar un camión para llevar una sola caja! Además, ¿sabías que es posible hacerlo sin recurrir al ciclo de vida `ngOnInit()`? Y no, no me refiero a lanzarlo en el constructor; eso es como ponerle una corbata a un gato, ¡una mala práctica en Angular! 😸

### 🪄 Aquí ocurre la magia queridos Devs…

```typescript
// ... otras importaciones
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-user-list',
  template: `
    <h2>Lista de Usuarios</h2>
    <ul>
      @for(user of users(); track user) {
        <li>{{ user.name }}</li>
      }
    </ul>
  `
})
export class UserListComponent {
    private userService = inject(UserService);
    public users = toSignal( this.userService.getUsers() );
}
```

¿No es simplemente **hermoso**? ✨ Hemos transformado unas cuantas líneas de código, incluso hemos usado la función [inject()](https://angular.dev/api/core/inject#) en lugar de inyectar el servicio en el constructor y, ¡vaya que se ve mucho mejor! Al utilizar `toSignal`, te libras de la molestia de manejar desuscripciones, porque este encantador operador se encarga de todo por ti. ¡Es como tener un asistente personal que se asegura de que todo funcione sin problemas! 🎉

### 🤔¿Qué pasa si mi observable no es un servicio http?

Vamos a crear un ejemplo usando el operador `Subject`. Tenemos un observable simple que convertimos en una señal. A través de un campo de texto y un evento, iremos actualizando el valor de esa señal de manera dinámica.

```typescript
// ... otras importaciones
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-state',
  template: `
    <h2>Estado: {{ state() }}</h2>
    <input type="text" #input/>
    <button (click)="changeValue(input.value)">Cambiar valor</button>
  `
})
export class StateComponent {
    private state$ = new Subject();
    public state = toSignal(this.state$);

    public changeValue(newState: string) {
        this.state$.next(newState);
    }
}
```

¿Te gustaría que tu estado empezara con un valor predeterminado? ¡Claro que sí! 🎯 Aunque el buen amigo `Subject` no lo puede hacer, el operador `toSignal` viene al rescate, permitiéndote establecer un valor inicial de forma súper fácil.

```typescript
// ... otras importaciones
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-state',
  template: `
    <h2>Estado: {{ state() }}</h2>
    <input type="text" #input/>
    <button (click)="changeValue(input.value)">Cambiar valor</button>
  `
})
export class StateComponent {
    private state$ = new Subject();
    public state = toSignal(this.state$, {
        initialValue: 'Hola mundo',
    });

    public changeValue(newState: string) {
        this.state$.next(newState);
    }
}
```

¿Quieres convertir observables que emitan un valor inicial? como por ejemplo el `BehaviorSubject`, en ese caso solo debes pasar el parámetro `requireSync` en `true`. Esto garantiza que la señal siempre tenga un valor y no `undefined`.

```typescript
// ... otras importaciones
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-state',
  template: `
    <h2>Estado: {{ state() }}</h2>
    <input type="text" #input/>
    <button (click)="changeValue(input.value)">Cambiar valor</button>
  `
})
export class StateComponent {
    private state$ = new BehaviorSubject('Primer valor');
    public state = toSignal(this.state$, {
        requireSync: true,
    });

    public changeValue(newState: string) {
        this.state$.next(newState);
    }
}
```

### Conclusiones

Este operador cuenta con más cualidades y casos de uso para sacarle todo el jugo, te invito a que lo revises, investigues un poco más y comiences a aplicarlo en tu código… ¡Sigamos escribiendo código limpio, escalable y mantenible en el tiempo! 🚀

### Referencias

* [https://angular.dev/api/core/rxjs-interop/toSignal?tab=description](https://angular.dev/api/core/rxjs-interop/toSignal?tab=description)
    
* [https://imaginaformacion.com/tutoriales/que-son-las-angular-signals-y-como-funcionan](https://imaginaformacion.com/tutoriales/que-son-las-angular-signals-y-como-funcionan)
    
* [https://angular.dev/api/core/inject#](https://angular.dev/api/core/inject#)
    
* [https://www.youtube.com/watch?v=hoYIUe\_e4Rs&ab\_channel=nicobytes](https://www.youtube.com/watch?v=hoYIUe_e4Rs&ab_channel=nicobytes)