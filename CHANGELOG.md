# Changelog

All notable changes to cloudflared.

## v1.0.0 (2026-10-28)

First tagged version, the baseline for moving cloudflared to tag deploys.

### Added
- **Lighthouse settings:** `x-lighthouse: {tier: infra}`. Lighthouse deploys cloudflared after sparkdb and before the apps, alone.
- The `spark` network: routes reach containers by name.

### Fixed
- Each deploy pulls the newest `cloudflare/cloudflared` image (`pull_policy: always`) and stops reusing an old local copy of `latest`.

### Changed
- The tunnel token comes from Cove as `CLOUDFLARED_CLOUDFLARE_TUNNEL_TOKEN`, following the key naming standard.
- `host.docker.internal` reaches the server itself, through `extra_hosts`.
