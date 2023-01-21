---
tableOfContents: true
---

# Examples

## Command-line interface (CLI)

Sometimes, you just want to spin up a local Hocuspocus instance really fast. Maybe just to give it a try, or to test your webhooks locally. Our CLI brings Hocuspocus to your command line in seconds.

Most likely you just want to run it with the npx command, although you can of course also install it globally or in your project with npm or yarn.

```bash
npx @hocuspocus/cli
npx @hocuspocus/cli --port 8080
npx @hocuspocus/cli --webhook http://localhost/webhooks/hocuspocus
npx @hocuspocus/cli --sqlite
```

## Express

Hocuspocus can be used with any WebSocket implementation that uses `ws` under the hood. When you don't call `listen()` on Hocuspocus, it will not start a WebSocket server itself but rather relies on you calling it's [`handleConnection()` method](/server/methods) manually.

To use Hocuspocus with [Express](https://expressjs.com), you need to use the `express-ws` package that adds WebSocket endpoints to Express applications. Then add a new WebSocket route and use Hocuspocus' `handleConnection()` method to do the rest.

```js
import express from "express";
import expressWebsockets from "express-ws";
import { Hocuspocus } from "@hocuspocus/server";

// Configure hocuspocus
const server = Hocuspocus.configure({
  // ...
});

// Setup your express instance using the express-ws extension
const { app } = expressWebsockets(express());

// A basic http route
app.get("/", (request, response) => {
  response.send("Hello World!");
});

// Add a websocket route for hocuspocus
// Note: make sure to include a parameter for the document name.
// You can set any contextual data like in the onConnect hook
// and pass it to the handleConnection method.
app.ws("/collaboration/:document", (websocket, request) => {
  const context = {
    user: {
      id: 1234,
      name: "Jane",
    },
  };

  server.handleConnection(websocket, request, request.params.document, context);
});

// Start the server
app.listen(1234, () => console.log("Listening on http://127.0.0.1:1234"));
```
		
IMPORTANT! Some extensions use the `onRequest`, `onUpgrade` and `onListen` hooks, that will not be fired in this scenario. Please read the docs of each extension on how to use them when integrating Hocuspocus in your framework of choice.


## Koa

```js
import Koa from "koa";
import websocket from "koa-easy-ws";
import { Hocuspocus } from "@hocuspocus/server";
import { Logger } from "@hocuspocus/extension-logger";

// Configure hocuspocus
const server = Hocuspocus.configure({
  // …
});

const app = new Koa();

// Setup your koa instance using the koa-easy-ws extension
app.use(websocket());

// Add a websocket route for hocuspocus
// Note: make sure to include a parameter for the document name.
// You can set any contextual data like in the onConnect hook
// and pass it to the handleConnection method.
app.use(async (ctx, next) => {
  const ws = await ctx.ws();
  const documentName = ctx.request.path.substring(1);

  server.handleConnection(
    ws,
    ctx.request,
    documentName,
    // additional data (optional)
    {
      user_id: 1234,
    }
  );
});

// Start the server
app.listen(1234);
```

IMPORTANT! Some extensions use the `onRequest`, `onUpgrade` and `onListen` hooks, that will not be fired in this scenario. Please read the docs of each extension on how to use them when integrating Hocuspocus in your framework of choice.


## Laravel (Draft)

We've created a Laravel package to make integrating Laravel and Hocuspocus seamless.

You can find details about it here: [ueberdosis/hocuspocus-laravel](https://github.com/ueberdosis/hocuspocus-laravel)
