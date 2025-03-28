---
title: Client options
sidebar_label: Options
sidebar_position: 2
slug: /client-options/
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## IO factory options

### `forceNew`

Default value: `false`

Whether to create a new Manager instance.

A Manager instance is in charge of the low-level connection to the server (established with HTTP long-polling or WebSocket). It handles the reconnection logic.

A Socket instance is the interface which is used to sends events to — and receive events from — the server. It belongs to a given [namespace](categories/06-Advanced/namespaces.md).

A single Manager can be attached to several Socket instances.

The following example will reuse the same Manager instance for the 3 Socket instances (one single WebSocket connection):

```js
const socket = io("https://example.com"); // the main namespace
const productSocket = io("https://example.com/product"); // the "product" namespace
const orderSocket = io("https://example.com/order"); // the "order" namespace
```

The following example will create 3 different Manager instances (and thus 3 distinct WebSocket connections):

```js
const socket = io("https://example.com"); // the main namespace
const productSocket = io("https://example.com/product", { forceNew: true }); // the "product" namespace
const orderSocket = io("https://example.com/order", { forceNew: true }); // the "order" namespace
```

Reusing an existing namespace will also create a new Manager each time:

```js
const socket1 = io(); // 1st manager
const socket2 = io(); // 2nd manager
const socket3 = io("/admin"); // reuse the 1st manager
const socket4 = io("/admin"); // 3rd manager
```

### `multiplex`

Default value: `true`

The opposite of `forceNew`: whether to reuse an existing Manager instance.

```js
const socket = io(); // 1st manager
const adminSocket = io("/admin", { multiplex: false }); // 2nd manager
```

## Low-level engine options

:::info

These settings will be shared by all Socket instances attached to the same Manager.

:::

### `addTrailingSlash`

*Added in v4.6.0*

The trailing slash which was added by default can now be disabled:

```js
import { io } from "socket.io-client";

const socket = io("https://example.com", {
  addTrailingSlash: false
});
```

In the example above, the request URL will be `https://example.com/socket.io` instead of `https://example.com/socket.io/`.


### `autoUnref`

*Added in v4.0.0*

Default value: `false`

With `autoUnref` set to `true`, the Socket.IO client will allow the program to exit if there is no other active timer/TCP socket in the event system (even if the client is connected):

```js
import { io } from "socket.io-client";

const socket = io({
  autoUnref: true
});
```

See also: https://nodejs.org/api/timers.html#timeoutunref


### `closeOnBeforeunload`

<details className="changelog">
    <summary>History</summary>

| Version | Changes                             |
|---------|-------------------------------------|
| v4.7.1  | The option now defaults to `false`. |
| v4.1.0  | First implementation.               |

</details>

Default value: `false`

Whether to (silently) close the connection when the [`beforeunload`](https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event) event is emitted in the browser.

When this option is set to `false` (the default value), the Socket instance will emit a `disconnect` event when the user reloads the page **on Firefox**:

![Example with Firefox when closeOnBeforeunload is set to false](/images/closeonbeforeunload-false.gif)

:::note

This behavior is specific to Firefox, on other browsers the Socket instance will not emit any `disconnect` event when the user reloads the page.

:::

When this option is set to `true`, all browsers will have the same behavior (no `disconnect` event when reloading the page):

![Example with Firefox when closeOnBeforeunload is set to true](/images/closeonbeforeunload-true.gif)

:::caution

If you use the `beforeunload` event in your application ("are you sure that you want to leave this page?"), it is recommended to leave this option to `false`.

:::

Please check [this issue](https://github.com/socketio/socket.io/issues/3639) for more information.

### `extraHeaders`

Default value: -

Additional headers (then found in `socket.handshake.headers` object on the server-side).

Example:

*Client*

```js
import { io } from "socket.io-client";

const socket = io({
  extraHeaders: {
    "my-custom-header": "1234"
  }
});
```

*Server*

```js
io.on("connection", (socket) => {
  console.log(socket.handshake.headers); // an object containing "my-custom-header": "1234"
});
```

:::caution

In a browser environment, the `extraHeaders` option will be ignored if you only enable the WebSocket transport, since the WebSocket API in the browser does not allow providing custom headers.

```js
import { io } from "socket.io-client";

const socket = io({
  transports: ["websocket"],
  extraHeaders: {
    "my-custom-header": "1234" // ignored
  }
});
```

This will work in Node.js or in React-Native though.

:::

Documentation: [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)


### `forceBase64`

Default value: `false`

Whether to force base64 encoding for binary content sent over WebSocket (always enabled for HTTP long-polling).


### `path`

Default value: `/socket.io/`

It is the name of the path that is captured on the server side.

:::caution

The server and the client values must match (unless you are using a path-rewriting proxy in between).

:::

*Client*

```js
import { io } from "socket.io-client";

const socket = io("https://example.com", {
  path: "/my-custom-path/"
});
```

*Server*

```js
import { createServer } from "http";
import { Server } from "socket.io";

const httpServer = createServer();
const io = new Server(httpServer, {
  path: "/my-custom-path/"
});
```

Please note that this is different from the path in the URI, which represents the [Namespace](categories/06-Advanced/namespaces.md).

Example:

```js
import { io } from "socket.io-client";

const socket = io("https://example.com/order", {
  path: "/my-custom-path/"
});
```

- the Socket instance is attached to the "order" Namespace
- the HTTP requests will look like: `GET https://example.com/my-custom-path/?EIO=4&transport=polling&t=ML4jUwU`


### `protocols`

*Added in v2.0.0*

Default value: -

Either a single protocol string or an array of protocol strings. These strings are used to indicate sub-protocols, so that a single server can implement multiple WebSocket sub-protocols (for example, you might want one server to be able to handle different types of interactions depending on the specified protocol).

```js
import { io } from "socket.io-client";

const socket = io({
  transports: ["websocket"],
  protocols: ["my-protocol-v1"]
});
```

Server:

```js
io.on("connection", (socket) => {
  const transport = socket.conn.transport;
  console.log(transport.socket.protocol); // prints "my-protocol-v1"
});
```

References:

- https://datatracker.ietf.org/doc/html/rfc6455#section-1.9
- https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket


### `query`

Default value: -

Additional query parameters (then found in `socket.handshake.query` object on the server-side).

Example:

*Client*

```js
import { io } from "socket.io-client";

const socket = io({
  query: {
    x: 42
  }
});
```

*Server*

```js
io.on("connection", (socket) => {
  console.log(socket.handshake.query); // prints { x: "42", EIO: "4", transport: "polling" }
});
```

The query parameters cannot be updated for the duration of the session, so changing the `query` on the client-side will only be effective when the current session gets closed and a new one is created:

```js
socket.io.on("reconnect_attempt", () => {
  socket.io.opts.query.x++;
});
```

Note: the following query parameters are reserved and can't be used in your application:

- `EIO`: the version of the protocol (currently, "4")
- `transport`: the transport name ("polling" or "websocket")
- `sid`: the session ID
- `j`: if the transport is polling but a JSONP response is required
- `t`: a hashed-timestamp used for cache-busting


### `rememberUpgrade`

Default value: `false`

If true and if the previous WebSocket connection to the server succeeded, the connection attempt will bypass the normal upgrade process and will initially try WebSocket. A connection attempt following a transport error will use the normal upgrade process. It is recommended you turn this on only when using SSL/TLS connections, or if you know that your network does not block websockets.


### `timestampParam`

Default value: `"t"`

The name of the query parameter to use as our timestamp key.


### `timestampRequests`

Default value: `true`

Whether to add the timestamp query param to each request (for cache busting).

### `transportOptions`

*Added in v2.0.0*

Default value: `{}`

Transport-specific options.

Example:

```js
import { io } from "socket.io-client";

const socket = io({
  path: "/path-for-http-long-polling/",
  transportOptions: {
    websocket: {
      path: "/path-for-websocket/"
    }
  }
});
```

### `transports`

<details className="changelog">
    <summary>History</summary>

| Version | Changes                                                 |
|---------|---------------------------------------------------------|
| v4.8.0  | You can now pass an array of transport implementations. |
| v4.7.0  | `webtransport` is added.                                |
| v1.0.0  | First implementation.                                   |

</details>

Default value: `["polling", "websocket", "webtransport"]`

The low-level connection to the Socket.IO server can either be established with:

- HTTP long-polling: successive HTTP requests (`POST` for writing, `GET` for reading)
- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [WebTransport](https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API)

The following example disables the HTTP long-polling transport:

```js
const socket = io("https://example.com", { transports: ["websocket"] });
```

Note: in that case, sticky sessions are not required on the server side (more information [here](categories/02-Server/using-multiple-nodes.md)).

By default, the HTTP long-polling connection is established first, and then an upgrade to WebSocket is attempted (explanation [here](categories/01-Documentation/how-it-works.md#upgrade-mechanism)). You can use WebSocket first with:

```js
const socket = io("https://example.com", {
  transports: ["websocket", "polling"] // use WebSocket first, if available
});

socket.on("connect_error", () => {
  // revert to classic upgrade
  socket.io.opts.transports = ["polling", "websocket"];
});
```

One possible downside is that the validity of your [CORS configuration](categories/02-Server/handling-cors.md) will only be checked if the WebSocket connection fails to be established.

You can also pass an array of transport implementations:

```js
import { io } from "socket.io-client";
import { Fetch, WebSocket } from "engine.io-client";

const socket = io({
  transports: [Fetch, WebSocket]
});
```

Here is the list of provided implementations:

| Transport       | Description                                                                                                                                              |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Fetch`         | HTTP long-polling based on the built-in [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch) method.                               |
| `NodeXHR`       | HTTP long-polling based on the `XMLHttpRequest` object provided by the [`xmlhttprequest-ssl`](https://www.npmjs.com/package/xmlhttprequest-ssl) package. |
| `XHR`           | HTTP long-polling based on the built-in [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) object.                      |
| `NodeWebSocket` | WebSocket transport based on the `WebSocket` object provided by the [`ws`](https://www.npmjs.com/package/ws) package.                                    |
| `WebSocket`     | WebSocket transport based on the built-in [`WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) object.                              |
| `WebTransport`  | WebTransport transport based on the built-in [`WebTransport`](https://developer.mozilla.org/en-US/docs/Web/API/WebTransport) object.                     |

Usage:

| Transport       | browser            | Node.js                | Deno               | Bun                |
|-----------------|--------------------|------------------------|--------------------|--------------------|
| `Fetch`         | :white_check_mark: | :white_check_mark: (1) | :white_check_mark: | :white_check_mark: |
| `NodeXHR`       |                    | :white_check_mark:     | :white_check_mark: | :white_check_mark: |
| `XHR`           | :white_check_mark: |                        |                    |                    |
| `NodeWebSocket` |                    | :white_check_mark:     | :white_check_mark: | :white_check_mark: |
| `WebSocket`     | :white_check_mark: | :white_check_mark: (2) | :white_check_mark: | :white_check_mark: |
| `WebTransport`  | :white_check_mark: | :white_check_mark:     |                    |                    |

(1) since [v18.0.0](https://nodejs.org/api/globals.html#fetch)
(2) since [v21.0.0](https://nodejs.org/api/globals.html#websocket)

### `tryAllTransports`

*Added in v4.8.0*

Default value: `false`

When setting the `tryAllTransports` option to `true`, if the first transport (usually, HTTP long-polling) fails, then the other transports will be tested too:

```js
import { io } from "socket.io-client";

const socket = io({
  tryAllTransports: true
});
```

This feature is useful in two cases:

- when HTTP long-polling is disabled on the server, or if CORS fails
- when WebSocket is tested first (with `transports: ["websocket", "polling"]`)

The only potential downside is that the connection attempt could take more time in case of failure, as there have been reports of WebSocket connection errors taking several seconds before being detected (that's one reason for using HTTP long-polling first). That's why the option defaults to `false` for now.


### `upgrade`

Default value: `true`

Whether the client should try to upgrade the transport from HTTP long-polling to something better.


### `withCredentials`

<details className="changelog">
    <summary>History</summary>

| Version | Changes                                                      |
|---------|--------------------------------------------------------------|
| v4.7.0  | The Node.js client now honors the `withCredentials` setting. |
| v3.0.0  | `withCredentials` now defaults to `false`.                   |
| v1.0.0  | First implementation.                                        |

</details>

Default value: `false`

Whether the cross-site requests should be sent including credentials such as cookies, authorization headers or TLS client certificates. Setting `withCredentials` has no effect on same-site requests.

```js
import { io } from "socket.io-client";

const socket = io("https://my-backend.com", {
  withCredentials: true
});
```

The server needs to send the right `Access-Control-Allow-* ` headers to allow the connection:

```js
import { createServer } from "http";
import { Server } from "socket.io";

const httpServer = createServer();
const io = new Server(httpServer, {
  cors: {
    origin: "https://my-frontend.com",
    credentials: true
  }
});
```

:::caution

You cannot use `origin: *` when setting `withCredentials` to `true`. This will trigger the following error:

> <i>Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at ‘.../socket.io/?EIO=4&transport=polling&t=NvQfU77’. (Reason: Credential is not supported if the CORS header ‘Access-Control-Allow-Origin’ is ‘*’)</i>

:::

Documentation:

- [XMLHttpRequest.withCredentials](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials)
- [Handling CORS](categories/02-Server/handling-cors.md)

:::info

Starting with version `4.7.0`, when setting the `withCredentials` option to `true`, the Node.js client will now include the cookies in the HTTP requests, making it easier to use it with cookie-based sticky sessions.

:::

### Node.js-specific options

The following options are supported:

- `agent`
- `pfx`
- `key`
- `passphrase`
- `cert`
- `ca`
- `ciphers`
- `rejectUnauthorized`

Please refer to the Node.js documentation:

- [tls.connect(options[, callback])](https://nodejs.org/dist/latest/docs/api/tls.html#tls_tls_connect_options_callback)
- [tls.createSecureContext([options])](https://nodejs.org/dist/latest/docs/api/tls.html#tls_tls_createsecurecontext_options)

Example with a self-signed certificate:

*Client*

```js
import { readFileSync } from "fs";
import { io } from "socket.io-client";

const socket = io("https://example.com", {
  ca: readFileSync("./cert.pem")
});
```

*Server*

```js
import { readFileSync } from "fs";
import { createServer } from "https";
import { Server } from "socket.io";

const httpServer = createServer({
  cert: readFileSync("./cert.pem"),
  key: readFileSync("./key.pem")
});
const io = new Server(httpServer);
```

Example with client-certificate authentication:

*Client*

```js
import { readFileSync } from "fs";
import { io } from "socket.io-client";

const socket = io("https://example.com", {
  ca: readFileSync("./server-cert.pem"),
  cert: readFileSync("./client-cert.pem"),
  key: readFileSync("./client-key.pem"),
});
```

*Server*

```js
import { readFileSync } from "fs";
import { createServer } from "https";
import { Server } from "socket.io";

const httpServer = createServer({
  cert: readFileSync("./server-cert.pem"),
  key: readFileSync("./server-key.pem"),
  requestCert: true,
  ca: [
    readFileSync("client-cert.pem")
  ]
});
const io = new Server(httpServer);
```

:::caution

`rejectUnauthorized` is a Node.js-only option, it will not bypass the security check in the browser:

![Security warning in the browser](/images/self-signed-certificate.png)

:::

## Manager options

:::info

These settings will be shared by all Socket instances attached to the same Manager.

:::

### `autoConnect`

Default value: `true`

Whether to automatically connect upon creation. If set to `false`, you need to manually connect:

```js
import { io } from "socket.io-client";

const socket = io({
  autoConnect: false
});

socket.connect();
// or
socket.io.open();
```


### `parser`

*Added in v2.2.0*

Default value: `require("socket.io-parser")`

The parser used to marshall/unmarshall packets. Please see [here](categories/06-Advanced/custom-parser.md) for more information.


### `randomizationFactor`

Default value: `0.5`

The randomization factor used when reconnecting (so that the clients do not reconnect at the exact same time after a server crash, for example).

Example with the default values:

- 1st reconnection attempt happens between 500 and 1500 ms (`1000 * 2^0 * (<something between -0.5 and 1.5>)`)
- 2nd reconnection attempt happens between 1000 and 3000 ms (`1000 * 2^1 * (<something between -0.5 and 1.5>)`)
- 3rd reconnection attempt happens between 2000 and 5000 ms (`1000 * 2^2 * (<something between -0.5 and 1.5>)`)
- next reconnection attempts happen after 5000 ms


### `reconnection`

Default value: `true`

Whether reconnection is enabled or not. If set to `false`, you need to manually reconnect:

```js
import { io } from "socket.io-client";

const socket = io({
  reconnection: false
});

const tryReconnect = () => {
  setTimeout(() => {
    socket.io.open((err) => {
      if (err) {
        tryReconnect();
      }
    });
  }, 2000);
}

socket.io.on("close", tryReconnect);
```

### `reconnectionAttempts`

Default value: `Infinity`

The number of reconnection attempts before giving up.

### `reconnectionDelay`

Default value: `1000`

The initial delay before reconnection in milliseconds (affected by the [randomizationFactor](#randomizationfactor) value).

### `reconnectionDelayMax`

Default value: `5000`

The maximum delay between two reconnection attempts. Each attempt increases the reconnection delay by 2x.


### `timeout`

Default value: `20000`

The timeout in milliseconds for each connection attempt.


## Socket options

:::info

These settings are specific to the given Socket instance.

:::

### `ackTimeout`

*Added in v4.6.0*

Default value: -

The default timeout in milliseconds used when waiting for an acknowledgement (not to be mixed up with the already existing [timeout](#timeout) option, which is used by the Manager during the connection).

Must be used in conjunction with [`retries`](#retries).

### `auth`

*Added in v3.0.0*

Default value: -

Credentials that are sent when accessing a namespace (see also [here](categories/02-Server/middlewares.md#sending-credentials)).

Example:

*Client*

```js
import { io } from "socket.io-client";

const socket = io({
  auth: {
    token: "abcd"
  }
});

// or with a function
const socket = io({
  auth: (cb) => {
    cb({ token: localStorage.token })
  }
});
```

*Server*

```js
io.on("connection", (socket) => {
  console.log(socket.handshake.auth); // prints { token: "abcd" }
});
```

You can update the `auth` map when the access to the Namespace is denied:

```js
socket.on("connect_error", (err) => {
  if (err.message === "invalid credentials") {
    socket.auth.token = "efgh";
    socket.connect();
  }
});
```

Or manually force the Socket instance to reconnect:

```js
socket.auth.token = "efgh";
socket.disconnect().connect();
```

### `retries`

*Added in v4.6.0*

Default value: -

The maximum number of retries. Above the limit, the packet will be discarded.

```js
const socket = io({
  retries: 3,
  ackTimeout: 10000
});

// implicit ack
socket.emit("my-event");

// explicit ack
socket.emit("my-event", (err, val) => { /* ... */ });

// custom timeout (in that case the ackTimeout is optional)
socket.timeout(5000).emit("my-event", (err, val) => { /* ... */ });
```

:::caution

The event must be acknowledged by the server (even with implicit ack):

```js
io.on("connection", (socket) => {
  socket.on("my-event", (cb) => {
    cb("got it");
  });
});
```

Else, the client will keep trying to send the event (up to `retries + 1` times).

:::
