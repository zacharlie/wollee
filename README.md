# wollee

Minimalist Wake-on-LAN management service, with a central server and lightweight downstream agents written in go.

## Rationale

wollee keeps power management simple: download the agent onto a downstream device and set it up as a service that sends heartbeats to an upstream server. The upstream server listens for heartbeats and keeps a centralized registry of connected clients. When a downstream client is unresponsive, the server can send a WoL packet to the client to wake it up.

You are responsible for the configuration of the other machines, configuring their

There are many frameworks which try to work across proxies, traverse subnets, manage system power, integrate with frameworks (e.g. homeassistant), or use fanciful protocol hacks to sense and control network devices.

## Components

- `cmd/server`: registry API, optional Telegram control, and embedded web UI.
- `cmd/agent`: heartbeat daemon that auto-detects local network identity.

## Configuration

Server configuration is file-based (`config.yaml`) and includes static host metadata:

```yaml
server:
  port: 8080
  subnetBroadcast: 192.168.1.255
  defaultHeartbeatInterval: 30s
  telegramToken: ""
  allowedTelegramUsers: []

hosts:
  - hostname: desktop
    mac: 00:11:22:33:44:55
```

`defaultHeartbeatInterval` is returned by `/register` and controls downstream heartbeat cadence.

## Running as applications

### Server

```bash
cp config.yaml.example config.yaml
task assets:sync
go run ./cmd/server run --config ./config.yaml
```

### Agent

```bash
go run ./cmd/agent run --upstream "http://server-a:8080,http://server-b:8080"
```

Agent options:

- `--upstream` (repeatable; comma/space delimited values supported)
- `--register-path` (default `/register`)
- `--request-timeout` (default `10s`)
- `--initial-heartbeat` (default `30s`)

The agent infers hostname, local IPv4, and matching interface MAC dynamically.

## Running as services

Server:

```bash
./wol-server install --config /absolute/path/config.yaml
./wol-server start
./wol-server stop
./wol-server uninstall
```

Agent:

```bash
./wol-agent install --upstream "http://server:8080"
./wol-agent start
./wol-agent stop
./wol-agent uninstall
```

## Telegram commands

If `server.telegramToken` is set and `allowedTelegramUsers` is non-empty:

- `/list`
- `/wake <hostname|mac>`

## Development

- `task lint`
- `task test`
- `task build:local`
- `task assets:sync`
- `task build:release`
