# Cross-version multiplayer

Horizon speaks its own protocol version. For remote servers that advertise a
different Minecraft protocol, the client can start a local ViaProxy process and
connect to it over loopback. ViaProxy translates between Horizon's protocol and
the backend server's protocol.

## Runtime behavior

- The client status-pings the server before connecting and reads its advertised
  protocol. Legacy status ping is attempted when the modern status handshake
  fails.
- A server already speaking Horizon's protocol is contacted directly.
- A different protocol is routed through a per-server ViaProxy process bound
  only to `127.0.0.1`. The backend version is auto-detected by ViaProxy.
- The process is reused while sessions/transfers need it and stopped after the
  final session closes. Leftover processes from a crashed client are reaped on
  the next cross-version join.
- The multiplayer list treats versions ViaProxy can translate as joinable.
- `HORIZON_CROSSVERSION=0` disables this behavior;
  `HORIZON_CROSSVERSION_ALWAYS=1` forces ViaProxy even for the current protocol;
  `HORIZON_CROSSVERSION_JAVA` can point to a Java executable.

## Requirements and authentication

ViaProxy 3.4.14 is downloaded on first use from its official GitHub release and
verified against a pinned SHA-256 digest before being stored in the game's
cross-version data directory. The downloaded jar is not bundled with Horizon.
ViaProxy is GPL-3.0; see https://github.com/ViaVersion/ViaProxy.

Java 17 or newer must be installed separately. The client searches
`HORIZON_CROSSVERSION_JAVA`, `JAVA_HOME`, `PATH`, and standard system Java
install locations. It does not execute Java from the game directory.

This integration explicitly configures ViaProxy with `auth-method NONE` and
frontend online mode disabled. It is for offline-mode servers. It does not
forge session signatures or bypass online-mode authentication; online-mode
servers remain unsupported through this route.

## Operational notes

The first cross-version join downloads approximately 49 MB and can take longer
while ViaProxy starts. Subsequent joins reuse the cached jar. Proxy logs can be
echoed to the game log with `HORIZON_CROSSVERSION_DEBUG=1`.

The integration code was adapted from the MIT-licensed
[JustKarkat/horizon-crossversion](https://github.com/JustKarkat/horizon-crossversion)
project. Its license is retained in
`crates/horizon-client/src/net/CROSSVERSION_LICENSE`.
