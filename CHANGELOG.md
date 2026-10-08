# Changelog

All notable changes to cloudflared. It deploys every commit on `main`, so changes are listed under **Unreleased** until a version is tagged.

## Unreleased

### Added
- **Lighthouse settings:** `x-lighthouse: {tier: infra}`. Lighthouse deploys cloudflared after sparkdb and before the apps, alone.
- The `spark` network: routes reach containers by name.

### Changed
- The tunnel token comes from Cove as `CLOUDFLARED_CLOUDFLARE_TUNNEL_TOKEN`, following the key naming standard.
- `host.docker.internal` reaches the server itself, through `extra_hosts`.
