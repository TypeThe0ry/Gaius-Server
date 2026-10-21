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
  server. A supplied target must exactly match that configured server.
* The plugin replies with `{"type":"connecting","host":...,"port":...,"targetConnectTimeoutMs":...}`
  and then `{"type":"connected","host":...,"port":...}` after the local
  socket is open. The browser may send `{"type":"flow","paused":true|false}`
  to control server-to-browser back-pressure.
* Binary WebSocket messages are copied to the local server, and bytes read
  from that socket are sent as binary WebSocket messages. Closing either side
  closes the other side.

The control messages and binary framing match Relay protocol version 1. The
plugin does not contain a game client, mappings, generated assets, or server
data; it only supplies the Paper-side transport adapter.

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

For an HTTPS browser page, terminate TLS at the deployment's reverse proxy and
use the corresponding `wss://` endpoint. Set `minecraft-host` to the local
bind address, keep `websocket-port` reachable from the browser, and restrict
`allowed-origins` to the page origins you operate. Set `access-token` only
when the browser and server share an out-of-band token; the sample
configuration contains no credential.

The plugin always checks that a requested target is the configured local
server. It enforces connection and frame-size limits, rejects unknown paths or
control messages, and closes an in-flight local socket when its browser
session ends.

## Development

Run the unit and integration tests with `mvn -B -ntp verify`. Tests use loopback
sockets and the reserved `example.com` documentation domain; they do not
contact a live server.
