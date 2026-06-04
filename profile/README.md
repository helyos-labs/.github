<div align="center">

<img alt="Helyos" src="assets/helyos-logo.png" width="200">

**Simple distributed container orchestration.**
**80% of Kubernetes use-cases with 20% of the complexity.**

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://github.com/helyos-labs/helyos/blob/main/LICENSE)
[![Built with Rust](https://img.shields.io/badge/built%20with-Rust-orange.svg)](https://www.rust-lang.org)

</div>

---

Helyos is a lightweight container orchestration platform built from the ground up in Rust. It is designed for teams that need multi-node container management, service discovery, and automatic TLS without the operational overhead of Kubernetes. A single binary daemon handles scheduling, health checking, networking, and secret management out of the box.

Where Kubernetes requires a constellation of control-plane components and years of expertise to operate well, Helyos ships as two binaries: `helyosd` (the daemon) and `helyos` (the CLI). Deploy services with familiar YAML specs, scale across nodes with a single join command, and let the built-in reverse proxy handle TLS certificates automatically. No etcd, no kubelet, no YAML-of-YAML.

Helyos supports both Docker and containerd as container runtimes, with automatic detection at startup. Its hexagonal architecture cleanly separates domain logic from infrastructure, making it straightforward to extend with new backends and adapters.

## Architecture

```
                         helyos (CLI)
                            |
                        HTTP REST API
                            |
    +--------------------------------------------------+
    |                    helyosd                          |
    |                                                  |
    |   Orchestrator (actor model)                     |
    |       |          |          |          |         |
    |   Scheduler   Health    Events    Secrets        |
    |       |       Checker   Watcher   (AES-256)      |
    |       |          |          |                     |
    |   +------+  +--------+  +-------+  +--------+   |
    |   |Docker|  |  SQLite |  | Proxy |  |Cluster |   |
    |   |contrd|  |  State  |  | nginx |  |  gRPC  |   |
    |   +------+  +--------+  | caddy |  +--------+   |
    |                          |traefik|               |
    |                          +-------+               |
    +--------------------------------------------------+
              |                           |
        helyos-core                    helyos-proxy
    (shared types & traits)    (HTTP/HTTPS reverse proxy)
```

## Repositories

| Repository | Description |
|:--|:--|
| [`helyos`](https://github.com/helyos-labs/helyos) | Meta-repository -- documentation, design specs, and implementation plans |
| [`helyos-core`](https://github.com/helyos-labs/helyos-core) | Core library -- domain types, port traits, actor-model orchestrator |
| [`helyosd`](https://github.com/helyos-labs/helyosd) | Daemon -- runtime adapters, REST API, cluster management, networking |
| [`helyos-cli`](https://github.com/helyos-labs/helyos-cli) | CLI tool -- deploy, scale, and manage containers from the terminal |
| [`helyos-proxy`](https://github.com/helyos-labs/helyos-proxy) | Reverse proxy -- HTTP/HTTPS routing with weighted load balancing |

## Quick Start

```bash
# Install Helyos
curl -sSfL https://raw.githubusercontent.com/helyos-labs/helyos/main/install.sh | sh

# Start the daemon
helyosd

# Deploy your first service
cat <<EOF > app.yaml
project: myapp
deployment:
  name: web
image: nginx:latest
replicas: 2
ports:
  - 8080
EOF

helyos deploy app.yaml
helyos status
```

## Key Features

- **Single binary daemon** -- no external dependencies beyond Docker or containerd
- **Multi-node clustering** -- join workers with a single token, automatic pod rescheduling on node failure
- **Built-in service discovery** -- embedded DNS server resolves `<service>.<project>.internal`
- **Automatic TLS** -- Let's Encrypt integration with certificate auto-renewal
- **Reverse proxy** -- nginx, Caddy, Traefik, or the built-in helyos-proxy as backends
- **Project isolation** -- logical namespaces with suspend, resume, and resource management
- **Encrypted secrets** -- AES-256-GCM at rest with per-node master keys
- **Health checking** -- HTTP, TCP, and exec probes with configurable thresholds
- **Weighted scheduling** -- spread or bin-pack strategies across heterogeneous nodes
- **WireGuard overlay** -- encrypted cross-node networking with automatic subnet allocation
- **Runtime flexibility** -- Docker and containerd support with automatic detection

## License

All Helyos repositories are licensed under [Apache-2.0](https://github.com/helyos-labs/helyos/blob/main/LICENSE).
