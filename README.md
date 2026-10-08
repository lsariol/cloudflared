# cloudflared

**The way in from the internet: a Cloudflare Tunnel for the `spark` server.**

cloudflared runs Cloudflare's tunnel connector. It opens an outbound connection to Cloudflare, so the server needs no open ports on the router: public traffic arrives through the tunnel and is handed to containers on the `spark` network by name.

- **No open router ports:** the tunnel only makes outbound connections.
- **Routes live in Cloudflare:** which public hostname goes to which container is set in the Cloudflare dashboard, not here.
- **One secret:** the tunnel token, from Cove.
- **Deployed by [Lighthouse](https://github.com/lsariol/lighthouse)** as infrastructure, on every push to `main`.

> **Documentation:** [DOCUMENTATION.md](DOCUMENTATION.md) covers configuration, routing a new hostname, deploying and troubleshooting. Changes: [CHANGELOG.md](CHANGELOG.md).

---

## Quick start

Lighthouse deploys it. By hand, on the server:

```bash
docker network inspect spark >/dev/null 2>&1 || docker network create spark
CLOUDFLARED_CLOUDFLARE_TUNNEL_TOKEN=<token> docker compose up -d
docker logs cloudflared       # "Registered tunnel connection" (several times)
```
