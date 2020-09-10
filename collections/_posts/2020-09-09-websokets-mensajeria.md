---
layout: post
title: Mensajería sencilla usando Web Sockets
---

Una aplicación de mensajería instántanea es relativamente fácil de programar, con el uso adecuado de
herramientas podemos tener nuestro propio sistema de mensajería en menos de 20 minutos.
El pequeño programa que te voy a mostrar aquí tiene sus problemas pero es un buen proyecto
didáctico para familiarizarte mejor con el uso de Web Sockets.

Si quieres tener una mirada a un proyecto más sencillo con Web Sockets, te recomiendo este [artículo]({{ site.baseurl }}/echo-server-nodejs).

El proyecto consiste de 2 clientes que pueden intercambiar mensajes mediante un servidor.
Si el cliente 1 manda un 'Hola' el servidor debe mandar ese mensaje al cliente 2 y viceversa.

## El servidor

Primero importemos los módulos que vamos a necesitar y definamos las cosas básicas,
estaremos usando el siempre 'express' para nuestro servidor, el módulo de node 'http' para integrarlo con ws,
el módulo 'url' para parsear la url que va a contener el 'id' del usuario y 
'ws' como librería de WebSocket.

```javascript
const express = require("express");
const http = require("http");
const url = require("url");
const WebSocket = require("ws");

const PORT = 3000;

let app = express();
let server = http.Server(app);

const wss = new WebSocket.Server({ server: server, path: "/echo" });
```

Como tenemos que mandar mensajes de un usuario a otro, tenemos que saber el socket de cada usuario,
para lograr este cometido simplemente usaremos id's de usuarios para identificar cada socket
y los guardaremos en un objeto de javascript para acceder fácilmente a ellos.

```javascript
let userSockets = {};

wss.on("connection", function connection(ws, request) {
  const parameters = url.parse(request.url, true);
  const userId = parameters.query.user_id;
  userSockets[userId] = ws;
  console.log("Conexion establecida para usuario", userId);

  ws.on("message", function incoming(message) {
    const { destinatarioId, mensaje } = JSON.parse(message);
    console.log("destinatario: %s, received: %s", destinatarioId, mensaje);
    const socketDestinatario = userSockets[destinatarioId];
    if (socketDestinatario) {
      socketDestinatario.send(message);
    } else {
      ws.send("el usuario no esta conectado");
    }
  });

  ws.send("Conexión con el servidor de eco exitosa");
});
```

- Primero definimos la variable 'userSockets' para almacenar los usuarios y sus respectivos sockets.
- Definimos lo que va a pasar en el evento 'connection' con una función de 2 parámetros.
- Parseamos la url para obtener el 'user\_id', guardamos el socket y mandamos un mensaje de confirmación.
- En la definición de lo que va a pasar en el evento 'message' obtenemos el destinatario y el mensaje.
- Obtenemos el socket del destinatario y en caso de no haberse conectado al servidor todavía,
mandamos un mensaje de error al emisor.

Finalmente hacemos que nuestra aplicación empieze a escuchar.

```javascript
server.listen(PORT, () => {
  console.log("Aplicacion corriendo en puerto:", PORT);
});
```

---

## El cliente

Para hacerlo sencillo utilizaremos 2 páginas html y les pondremos javascript para
mandar un mensaje, como prueba de concepto, luego podrás generalizar el uso agregando
botones, cuadros de texto, etc.
Para este proyecto únicamente veremos la generalización, es decir solo veremos los mensajes en consola.

Primero creamos 2 documentos html: index.html e index2.html, dentro utilizaremos el tag 'script'
para hacer la conexión con javascript

'index.html'

```javascript
  const echoSocketUrl = 'ws://localhost:3000/echo?user_id=1'
  const socket = new WebSocket(echoSocketUrl);

  socket.onopen = () => {
    const mensaje = { destinatarioId: 2, mensaje: "hola"}
    socket.send(JSON.stringify(mensaje)); 
  }

  socket.onmessage = e => {
    console.log('Message from server:', e.data);
  }
```
Nótese que definiendo la url agregamos el parámetro 'user\_id=1' y en el mensaje pasamos el id del destinatario.

'index2.html'

```javascript
  const echoSocketUrl = 'ws://localhost:3000/echo?user_id=2'
  const socket = new WebSocket(echoSocketUrl);

  socket.onopen = () => {
    const mensaje = { destinatarioId: 1, mensaje: "hola"}
    socket.send(JSON.stringify(mensaje)); 
  }

  socket.onmessage = e => {
    console.log('Message from server:', e.data);
  }
```

Y listo, ya tenemos un sistema de mensajería básico. Abre los 2 documentos html en un navegador
y revisa las respuestas en la consola.

Como ya tenemos la versión general de un sistema de mensajería, lo único que queda es mejorarla.
Lo primero que podemos hacer para mejorarla es hacer que los mensajes no sean estáticos,
podríamos tener un cuadro de texto y un botón de envío para mandar diferentes mensajes y
también podemos agregar un cuadro de texto que tenga el historial de mensajes.

Por el lado del servidor también podemos agregar mejoras, una de las mejoras necesarias
para que la aplicación no consuma memoria innecesaria sería
el eliminar de memoria los sockets que no se están usando, otra mejora sería el separar
los eventos de 'message' y 'connection', y por último intenta hacer seguro el intercambio de
mensajes.
