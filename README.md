# Gaius Server Bridge

Gaius Server Bridge is a Paper plugin that bridges a browser WebSocket client
to the local Minecraft server socket. It accepts one WebSocket tunnel per
browser session and forwards the server protocol bytes without translating
them or exposing an arbitrary TCP proxy.

The plugin implements Relay protocol version 1. The browser can therefore use
the same direct-server probe and fallback rules as a RelayNode. The public
transport contract is deliberately small:

* WebSocket clients connect to `/tunnel` and must satisfy `allowed-origins`.
* The first text message is `{"type":"connect"}`. It may include `host`,
  `port`, and `token`; omitted `host` and `port` mean the configured local
  server. A supplied `host` is the address the browser used to reach this
  machine and is echoed back unchanged; a supplied `port` must equal the
  configured server's port. The plugin always dials only the configured
  local server.
* The plugin replies with `{"type":"connecting","host":...,"port":...,"targetConnectTimeoutMs":...}`
  and then `{"type":"connected","host":...,"port":...}` after the local
  socket is open. The browser may send `{"type":"flow","paused":true|false}`
  to control server-to-browser back-pressure.
* Binary WebSocket messages are copied to the local server, and bytes read
  from that socket are sent as binary WebSocket messages. Closing either side
  closes the other side.

The control messages and binary framing match Relay protocol version 1. The
plugin does not contain a game client or server data; it only supplies the
Paper-side transport adapter.

## Build

Use JDK 21 and Maven from this repository:

```sh
mvn -B -ntp verify
```

The shaded plugin is written to `target/gaius-server-plugin-<version>.jar`.
The Paper API is provided at runtime, while the WebSocket and JSON libraries
are included in that archive.

## Install and configure

Copy the shaded archive to the Paper server's `plugins/` directory and restart
the server. The default endpoint is:

```text
ws://example.com:8081/tunnel
```

The browser client looks for the plugin on port 8081 of the host typed in the
server list, so keep the default `websocket-port: 8081` unless every player
opens the client with `?directPluginPort=<port>`. Players join with the normal
game address (`<host>:<server-port>`), not the WebSocket port. The client uses
offline-mode profiles, so set `online-mode=false` in `server.properties`.

For an HTTPS browser page, terminate TLS at the deployment's reverse proxy and
use the corresponding `wss://` endpoint. Set `minecraft-host` to the local
bind address, keep `websocket-port` reachable from the browser, and restrict
`allowed-origins` to the page origins you operate. Set `access-token` only
when the browser and server share an out-of-band token; the sample
configuration contains no credential.

The plugin always checks that a requested port is the configured local
server's port. It enforces connection and frame-size limits, rejects unknown paths or
control messages, and closes an in-flight local socket when its browser
session ends.

## Development

Run the unit and integration tests with `mvn -B -ntp verify`. Tests use loopback
sockets and the reserved `example.com` documentation domain; they do not
contact a live server.
