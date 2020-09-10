---
layout: post
title: Ejemplo de un servidor eco en nodejs usando Web Sockets (Echo Server Nodejs)
---

Un servidor de eco es una aplicación que permite la conexión entre cliente y servidor
de forma tal que el cliente pueda mandar un mensaje al servidor y el servidor reciba
el mensaje y reenviarlo (hacer eco) al cliente.

El nombre es autodescriptivo, no es un concepto complicado. Si el cliente manda 'Hola',
el servidor recibe 'Hola' y manda 'Hola' al cliente. 
El proyecto es sencillo y podemos usarlo para aprender acerca de Web Sockets, bien empecemos.

Lo primero que necesitamos es crear el proyecto con node:
```bash
$ mkdir echo-server && cd echo-server && npm init -y
```
Ahora instalemos las dependencias, utilizaremos la librería 'ws' por ser bastante transparente
y el siempre confiable 'express' como servidor.

```bash
$ npm i express ws
```

Crearemos el punto de entrada de la aplicación 'index.js' y adentro vamos a definir nuestro servidor.


Lo primero que haremos es importar todas las librerías que necesitaremos.
Primero el siempre confiable 'express', también necesitamos el módulo 'http' de node y
finalmente la librería de WebSocket 'ws'.

```javascript
const express = require('express')
const http = require('http')
const WebSocket = require('ws')
```

Ahora definamos el puerto e inicializamos el servidor express, luego inicializamos el servidor de 'http'.

```javascript
const PORT = 3000

let app = express()
let server = http.Server(app)
```

Ahora inicializamos el WebSocket, como argumento le pasamos un objeto con los parametros 'server' y 'path'.
Las conexiones que llegan al servidor las manejaremos mediante eventos, 'on connection' es el evento que se dispara
cuando el servidor obtiene una conexión; el evento 'on message' se dispara cuando el 'socket' recibe un mensaje.

```javascript
const wss = new WebSocket.Server({ server: server, path: '/echo'  })

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
    ws.send(message)
  });
 
  ws.send('Conexión con el servidor de eco exitosa');
});
```
Finalmente iniciamos a escuchar en el servidor.

```javascript
server.listen(PORT, () => {
  console.log("Aplicacion corriendo en puerto:", PORT)
})
```

Este es todo el código.

```javascript
const express = require('express')
const http = require('http')
const WebSocket = require('ws')

const PORT = 3000

let app = express()
let server = http.Server(app)

const wss = new WebSocket.Server({ server: server, path: '/echo'  })

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
    ws.send(message)
  });
 
  ws.send('Conexión con el servidor de eco exitosa');
});

server.listen(PORT, () => {
  console.log("Aplicacion corriendo en puerto:", PORT)
})

```

Ahora que ya tenemos nuestro servidor, tenemos que escribir un cliente.
Para hacer esto solo necesitamos una página html y algo de javascript.
Creamos un documento 'index.html', lo único que importa de este documento
es el javascript, puedes anexarlo como un documento separado con el tag script:src
o directamente escribir en un tag script.

```javascript
const echoSocketUrl = 'ws//localhost:3000/echo/'
const socket = new WebSocket(echoSocketUrl);

socket.onopen = () => {
  socket.send('Mensaje urgente!'); 
}

socket.onmessage = e => {
  console.log('Mensaje del servidor:', e.data);
}
```

Finalmente solo abriremos el documento index.html en un navegador y revisamos la consola para ver que está funcionando correctamente.

Después de acabar este pequeño proyecto te habrás dado cuenta de que no es muy útil, solo estamos repitiendo un mensaje enviado al mismo cliente
en un post subsecuente haremos cosas más útiles con Web Sockets, algo como mandar un mensaje de un cliente a otro.
