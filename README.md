# Infrastructure Documentation

Comprehensive documentation for the reverb256 NixOS homelab cluster.

## Quick Start

```bash
# View the full scoping audit plan
cat scoping-audit-plan.md
```

## Documentation

- **[Scoping Audit Plan](scoping-audit-plan.md)** — Comprehensive plan for repository organization, infrastructure mapping, security posture, and technical debt remediation.

## Cluster Overview

- **Nodes:** 6 NixOS hosts (zephyr, nexus, forge, sentry, krash3, krash1.5)
- **Cluster:** k3s Kubernetes (3 control plane, 2 agents, 1 planned)
- **VIP:** 10.1.1.100 (HA via Keepalived)
- **Storage:** Garage S3 distributed object storage
- **Monitoring:** Prometheus + Grafana + Alertmanager

## Key Repositories

- [nixos-config](https://github.com/reverb256/nixos-config) — NixOS cluster configuration
- [homelab-ops](https://github.com/reverb256/homelab-ops) — Operational scripts and runbooks
- [hermes-skills](https://github.com/reverb256/hermes-skills) — Hermes agent skills
- [maplespike](https://github.com/reverb256/maplespike) — MapleSpike product (monorepo)

## License

MIT
