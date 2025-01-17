---
title: "Guía práctica: Implementa caché con Redis en tu aplicación Express."
description: "Mejora el rendimiento de tus aplicaciones Node.js gestionando el caché HTTP con Redis."
date: "Jan 16 2025"
---

### Configuraciones iniciales

Para comenzar, ubícate en el directorio de tu preferencia y crea un nuevo proyecto de `Node.js` desde cero.

```bash
mkdir cache-node-redis && cd cache-node-redis
```

Luego, inicializa el proyecto con el siguiente comando:

```bash
npm init -y
```

A continuación, instala las dependencias necesarias para el proyecto:

```bash
npm install express axios redis response-time
```

Estas dependencias cumplen los siguientes propósitos:

* `axios`: Para realizar solicitudes HTTP a una API externa. En este ejercicio, utilizaremos la [API de Rick and Mo](https://rickandmortyapi.com/)[rty.](https://rickandmortyapi.com/)
    
* `express`: Para configurar un servidor web ligero y eficiente.
    
* [`redis`:](https://rickandmortyapi.com/) Para manejar la caché de nuestras peticiones.
    
* `response-time`: Para medir el tiempo de respuesta de las solicitudes antes y después de implementar el almacenamiento en caché.
    

Después de ejecutar los comandos anteriores, abre el proyecto en Visual Studio Code o tu editor favorito y realiza el siguiente ajuste en el archivo `package.json`:

```json
{
  "type": "module" // ➡️ Configuración necesaria para usar ES Modules
}
```

### Configurando el servidor de Redis con Docker

Si deseas instalar `Redis`, puedes usar distintas opciones dependiendo de tu sistema operativo. Para este ejemplo, utilizaremos `Docker`.

Crea un archivo `docker-compose.yml` en la raíz del proyecto con la siguiente configuración:

```yaml
version: '3.7'

services:
  redis:
    image: redis
    ports:
      - "6379:6379"
```

Luego, ejecuta el siguiente comando para levantar el servidor de `Redis`:

```bash
docker compose up -d
```

### Desarrollando nuestra API

Crea un archivo `index.js` y configura un servidor básico con `Express`. Agregaremos un endpoint de tipo `GET` para obtener los personajes de la API de Rick and Morty.

```javascript
import express from 'express'
import axios from 'axios'

const app = express()
const port = 3000
const route = 'characters'

app.get(`/${route}`, async (req, res) => {
  const { data } = await axios.get('https://rickandmortyapi.com/api/character')
  return res.json(data)
})

app.listen(port, () => {
  console.log(`El servidor está corriendo en el puerto ${port}`)
})
```

Este código realiza una solicitud HTTP a la API externa y retorna los datos directamente. El siguiente paso será implementar `Redis` para almacenar los resultados en caché y reducir los tiempos de respuesta.

### Integrando Redis como caché

1. **Configurando el cliente de Redis**  
    Importa la función `createClient` desde el paquete `redis` y conéctala al servidor local:
    
    ```javascript
    import { createClient } from 'redis'
    
    const redisClient = createClient({
      host: '127.0.0.1',
      port: 6379
    })
    await redisClient.connect()
    ```
    

2. **Almacenando y recuperando datos en caché**  
    Modifica el endpoint para usar `Redis`. Si los datos ya están en caché, los retornará directamente. De lo contrario, obtendrá los datos desde la API externa y los almacenará en el caché:
    
    ```javascript
    app.get(`/${route}`, async (req, res) => {
      const cache = await redisClient.get(route)
      if (cache) return res.json(JSON.parse(cache)) // Recuperar del caché
    
      const { data } = await axios.get('https://rickandmortyapi.com/api/character')
      await redisClient.set(route, JSON.stringify(data)) // Almacenar en caché
      return res.json(data)
    })
    ```
    
3. **Midiendo el tiempo de respuesta**  
    Usa el middleware `response-time` para medir cuánto tardan las solicitudes:
    
    ```javascript
    import responseTime from 'response-time'
    app.use(responseTime())
    ```
    

4. **Archivo completo**  
    Tu archivo `index.js` final debería verse así:  
    Lo siguiente que haremos es guardar en caché la respuesta de la API cuando se soliciten los datos por primera vez, en las veces posteriores preguntaremos si existen datos en caché, de ser así, devolveremos la información que se encuentra en caché en lugar de realizar una nueva solicitud a la API externa.
    
    ```javascript
    import express from 'express'
    import responseTime from 'response-time'
    import axios from 'axios'
    import { createClient } from 'redis'
    
    const app = express()
    const port = 3000
    const redisClient = createClient({
      host: '127.0.0.1',
      port: 6379
    })
    const route = 'characters'
    
    app.use(responseTime())
    
    app.get(`/${route}`, async (req, res) => {
      const cache = await redisClient.get(route)
      if (cache) return res.json(JSON.parse(cache))
    
      const { data } = await axios.get('https://rickandmortyapi.com/api/character')
      await redisClient.set(route, JSON.stringify(data))
      return res.json(data)
    })
    
    await redisClient.connect()
    
    app.listen(port, () => {
      console.log(`El servidor está corriendo en el puerto ${port}`)
    })
    ```
    

### Probando nuestra API

Inicia el servidor y accede a [`http://localhost:3000/characters`](http://localhost:3000/characters).

* **Primera solicitud**: Como no hay datos en caché, el tiempo de respuesta será mayor, aproximadamente **357 ms**.
    
* **Solicitudes posteriores**: Al reutilizar datos del caché, el tiempo de respuesta se reduce a tan solo **3 ms**.
    
![Cache before](/cache-before.png)

![Cache after](/cache-after.png)

Con esta implementación, tu aplicación ahora es más eficiente al reducir la carga en la API externa y optimizar el tiempo de respuesta gracias al almacenamiento en caché con `Redis`. 🚀

### Referencias:

* [https://axios-http.com/docs/intro](https://axios-http.com/docs/intro)
    
* [https://expressjs.com/es/](https://expressjs.com/es/)
    
* [https://hub.docker.com/\_/redis](https://hub.docker.com/_/redis)
    
* [https://www.npmjs.com/package/response-time](https://www.npmjs.com/package/response-time)
    
* [https://youtu.be/TazYA-wqOkY?si=ef096a\_71m3Y9Ke7](https://youtu.be/TazYA-wqOkY?si=ef096a_71m3Y9Ke7)