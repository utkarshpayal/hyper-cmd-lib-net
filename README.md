# hyper-cmd-lib-net

Small Node.js helper library ("HyperCmd Net lib") for piping data between two duplex streams/sockets, with optional gzip compression and connection counters. It is used by the HyperCmd tooling.

This repository is a fork of [holepunchto/hyper-cmd-lib-net](https://github.com/holepunchto/hyper-cmd-lib-net).

## Features / API

`index.js` exports two functions:

### `connPiper(connection, dst, opts = {}, stats = {})`

Pipes data in both directions between `connection` and the stream returned by calling `dst()`.

- If `dst()` returns `null`, `connection` is destroyed and `stats.rejectCnt` is incremented.
- Otherwise `stats.locCnt` and `stats.remCnt` are incremented, and decremented again when the pipe is torn down.
- `end` on either side ends the other; `error` or `close` on either side destroys both.
- Options:
  - `compress` (boolean): gzip/gunzip each chunk using `node:zlib`. Use together with `isServer`.
  - `isServer` (boolean): selects the direction of compression. On the server side, data from the local stream is gzipped before being written to `connection`, and data from `connection` is gunzipped. The client side does the reverse.
  - `debug` (boolean): log when the local stream emits `connect`.
  - `onDestroy(err)` (function): called once when the pipe is destroyed.

### `connRemoteCtrl(connection, opts = {}, stats = {})`

Wraps a connection for sending control data.

- Increments `stats.remCnt` (decremented on destroy).
- Destroys the connection on `error` or `close`, calling `opts.onDestroy(err)` if provided; logs errors when `opts.debug` is set.
- Returns `{ send(data) }`, which writes to the connection unless it has already been destroyed.

## Tech stack

- Node.js (CommonJS), using only the built-in `node:zlib` module. There are no npm dependencies.

## Installation

```sh
git clone https://github.com/utkarshpayal/hyper-cmd-lib-net.git
cd hyper-cmd-lib-net
```

## Usage

```js
const net = require('net')
const { connPiper } = require('./index.js') // or require('hyper-cmd-lib-net') when installed as a package

const stats = {}

// Simple TCP forwarder: every incoming connection on 8080 is piped to localhost:3000
net.createServer(socket => {
  connPiper(socket, () => net.connect(3000, 'localhost'), {}, stats)
}).listen(8080)
```

## Tests / build

There is no build step and no test suite in this repository.

## License

MIT, as declared in `package.json` (no separate `LICENSE` file is included).
