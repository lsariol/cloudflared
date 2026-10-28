# cloudflared Documentation

The reference for **cloudflared**, the Cloudflare Tunnel connector that carries all public traffic into the `spark` server.

- [README](README.md): a short overview.
- [Lighthouse](https://github.com/lsariol/lighthouse): deploys it. Its `DOCUMENTATION.md` §7 says what a project needs to be reachable by name on `spark`.
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/): Cloudflare's own documentation.

---

## Contents

1. [Overview](#1-overview)
2. [Configuration](#2-configuration)
3. [Routing a hostname to a project](#3-routing-a-hostname-to-a-project)
4. [Deploying](#4-deploying)
5. [Operations](#5-operations)
6. [Troubleshooting](#6-troubleshooting)
7. [Known limitations](#7-known-limitations)

---

## 1. Overview

```
 browser ──► Cloudflare (DNS, TLS) ──► tunnel ──► cloudflared ──► http://<container>:<port> on spark
```

| Item | Value |
|---|---|
| Image | `cloudflare/cloudflared:latest`, pulled fresh on every deploy (`pull_policy: always`) |
| Container | `cloudflared`, compose project `cloudflared` |
| Command | `tunnel --no-autoupdate run` (a tunnel managed in the Cloudflare dashboard) |
| Network | `spark`, so it reaches containers by name |
| Also reaches | The server itself, as `host.docker.internal` |
| Deployed by | Lighthouse, on every push to `main`: `tier: infra` |

The tunnel connects outwards to Cloudflare. Nothing on the router is forwarded, and the server's public address isn't exposed.

---

## 2. Configuration

| Setting | Value | Notes |
|---|---|---|
| `TUNNEL_TOKEN` | `${CLOUDFLARED_CLOUDFLARE_TUNNEL_TOKEN}` | From Cove. Identifies the tunnel; anyone holding it can run a connector for it |
| `extra_hosts` | `host.docker.internal:host-gateway` | Lets a route point at a port published on the server itself |

The tunnel's routes (public hostnames and where they go) are stored in Cloudflare, not in this repository.

---

## 3. Routing a hostname to a project

1. **The project** joins the `spark` network and has a name nobody else on `spark` uses: its `container_name` or service name (Lighthouse §7, rule 9). It doesn't need to publish a port.
2. **In the Cloudflare dashboard** (Zero Trust → Networks → Tunnels → this tunnel → Public Hostname), add the hostname with the service `http://<container name>:<port>`.
3. **Check:** the hostname loads. If not, `docker logs cloudflared` shows the request and why it failed.

Prefer a container name over `host.docker.internal`: it doesn't need a published port, so the project isn't reachable from the home network around Cloudflare.

---

## 4. Deploying

Lighthouse deploys every push to `main`, pulling the newest cloudflared image each time. As `tier: infra`, it goes after sparkdb and before the apps, alone. The swap drops open connections for a moment; the tunnel reconnects within seconds.

**Changing the token:** create a new one in the Cloudflare dashboard, `update CLOUDFLARED_CLOUDFLARE_TUNNEL_TOKEN <token>` in Cove, then `lh deploy cloudflared`.

---

## 5. Operations

| Task | Command (on the server) |
|---|---|
| Is the tunnel up? | `docker logs --tail 20 cloudflared`; the Cloudflare dashboard shows the connector as healthy |
| Restart it | `lh restart cloudflared:cloudflared` |
| Redeploy | `lh deploy cloudflared` |

---

## 6. Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Every hostname shows a Cloudflare error page | The connector is down: `docker ps` for `cloudflared`, then `docker logs cloudflared` |
| `Unauthorized: Invalid tunnel secret` or similar at start | The token in Cove is wrong or was rotated. Update it and redeploy |
| One hostname fails (502 or 504) | Its route points at a name or port that doesn't answer: check the container is on `spark` and the name and port in the route |
| A route reaches the wrong project | Two containers answer to the same name on `spark`; Lighthouse refuses that on deploy, so check containers started by hand |

---

## 7. Known limitations

1. **The image is `latest`, not pinned:** every deploy pulls the newest cloudflared version, so a redeploy of the same tag can run a different cloudflared. Pin it (e.g. `cloudflare/cloudflared:2026.x.y`) if that ever causes trouble.
2. **Routes aren't in git:** they live in the Cloudflare dashboard.
3. **No healthcheck:** Lighthouse only checks that the container stays running.
