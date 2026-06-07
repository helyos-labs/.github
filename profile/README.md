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

The control plane is secure by default: the daemon serves an HTTPS REST API with multi-user API tokens, generating a self-signed certificate and a CLI context on first run so local use needs zero configuration. Point `helyos` at a remote cluster with `helyos login <server>`, which pins the daemon's CA and stores a named, kubectl-style connection context.

Helyos supports both Docker and containerd as container runtimes, with automatic detection at startup. Its hexagonal architecture cleanly separates domain logic from infrastructure, making it straightforward to extend with new backends and adapters.

## Quick Start

```bash
# Install Helyos
curl -sSfL https://raw.githubusercontent.com/helyos-labs/helyos/main/install.sh | sh

# Start the daemon -- it auto-generates an API token, a self-signed
# certificate, and a local CLI context on first run
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

# Talk to a remote cluster: pin its CA and store a named context
helyos login https://cluster.example.com:6443
```

## Key Features

- **Single binary daemon** -- no external dependencies beyond Docker or containerd
- **Secure-by-default control plane** -- HTTPS REST API, multi-user API tokens, self-signed cert auto-generated (or bring your own)
- **kubectl-style remote control** -- `helyos login` pins the daemon CA and stores named connection contexts
- **Multi-node clustering** -- join workers with a single token, automatic pod rescheduling on node failure
- **Built-in service discovery** -- embedded DNS server resolves `<deployment>.<project>.internal`
- **Automatic TLS for services** -- ACME (Let's Encrypt) provisioning with certificate auto-renewal
- **Reverse proxy** -- nginx, Caddy, Traefik as backends
- **Project isolation** -- logical namespaces with suspend, resume, and resource management
- **Encrypted secrets** -- AES-256-GCM at rest with per-node master keys
- **Health checking** -- HTTP probes with configurable thresholds and automatic restart
- **Weighted scheduling** -- spread or bin-pack strategies across heterogeneous nodes
- **WireGuard overlay** (experimental) -- encrypted cross-node networking with automatic subnet allocation
- **Runtime flexibility** -- Docker and containerd support with automatic detection

## License

All Helyos repositories are licensed under [Apache-2.0](https://github.com/helyos-labs/helyos/blob/main/LICENSE).
