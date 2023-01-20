---
tableOfContents: true
---

# Hooks

## Introduction

Hocuspocus offers hooks to extend it's functionality and integrate it into existing applications. Hooks are configured as simple methods the same way as [other configuration options](/guide/configuration) are.

Hooks accept a hook payload as first argument. The payload is an object that contains data you can use and manipulate, allowing you to built complex things on top of this simple mechanic.

Hooks are required to return a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), the easiest way to do that is to mark the function as `async` (Node.js version must 14+). In this way, you can do things like executing API requests, running DB queries, trigger webhooks or whatever you need to do to integrate it into your application.

## Lifecycle

Hooks will be called on different stages of the Hocuspocus lifecycle. For example the `onListen` hook will be called when you call the `listen()` method on the server instance.

Some hooks allow you not only to react to those events but also to intercept them. For example the `onConnect` hook will be fired when a new connection is made to underlying websocket server. By rejecting the Promise in your hook (or throwing an empty exception if using async) you can terminate the connection and stop the chain.

## The hook chain

Extensions use hooks to add additional functionality to Hocuspocus. They will be called after another in the order of their registration with your configuration as the last part of the chain.

If the Promise in a hook is rejected it will not be called for the following extensions or your configuration. It's like a stack of middlewares a request has to go through. Keep that in mind when working with hooks.

By way of illustration, if a user isn’t allowed to connect: Just send `reject()` in the `onConnect()` hook. Nice, isn’t it?



## Available hooks

| Hook                  | Description                               | Link                                          |
|-----------------------|-------------------------------------------|-----------------------------------------------|
| `beforeHandleMessage` | Before handling a message                 | [Read more](/api/hooks/before-handle-message) |
| `onConnect`           | When a connection is established          | [Read more](/api/hooks/on-connect)            |
| `connected`           | After a connection has been establied     | [Read more](/api/hooks/connected)             |
| `onAuthenticate`      | When authentication is passed             | [Read more](/api/hooks/on-authenticate)       |
| `onAwarenessUpdate`   | When awareness changed                    | [Read more](/api/hooks/on-awareness-update)   |
| `onLoadDocument`      | When a new document is created            | [Read more](/api/hooks/on-load-document)      |
| `onChange`            | When a document has changed               | [Read more](/api/hooks/on-change)             |
| `onDisconnect`        | When a connection was closed              | [Read more](/api/hooks/on-disconnect)         |
| `onListen`            | When the serer is intialized              | [Read more](/api/hooks/on-listen)             |
| `onDestroy`           | When the server will be destroyed         | [Read more](/api/hooks/on-destroy)            |
| `onConfigure`         | When the server has been configured       | [Read more](/api/hooks/on-configure)          |
| `onRequest`           | When a HTTP request comes in              | [Read more](/api/hooks/on-request)            |
| `onStoreDocument`     | When a document has been changed          | [Read more](/api/hooks/on-store-document)     |
| `onUpgrade`           | When the WebSocket connection is upgraded | [Read more](/api/hooks/on-upgrade)            |

## Usage

```js
import { Server } from '@hocuspocus/server'

const server = Server.configure({
  async onAuthenticate({ documentName, token }) {

    // Could be an API call, DB query or whatever …
    // The endpoint should return 200 OK in case the user is authenticated, and an http error
    // in case the user is not.
    return axios.get('/user', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    })
  },
})

server.listen()
```
