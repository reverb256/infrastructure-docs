# Shell Aliases & Reference

Cluster shell aliases (managed via Home Manager `modules/fish.nix`). All hosts run **fish** (`/run/current-system/sw/bin/fish`).

## Host Access (mosh-first)

| Alias | Command | Fallback |
|-------|---------|----------|
| `zephyr` | `mosh j_kro@zephyr` | `zssh` |
| `nexus` | `mosh j_kro@nexus` | `nssh` |
| `sentry` | `mosh j_kro@sentry` | `sssh` |
| `forge` | `mosh j_kro@forge` | `fssh` |

Mosh keeps sessions alive through network drops, IP changes, sleep/wake. SSH fallbacks for when mosh UDP is blocked.

## Deploy

| Alias | Command |
|-------|---------|
| `ndeploy` | commit + push + colmena apply |
| `ndeploy-z` / `-n` / `-s` | apply single host |

## Status

| Alias | Command |
|-------|---------|
| `tailstat` | `tailscale status` |
| `tailver` | `tailscale version` |
| `niri-leak` | check NVIDIA mapping count |
| `niri-restart` | `niri msg action quit` |
| `kgp` | `kubectl get pods -A` |
| `kgn` | `kubectl get nodes -o wide` |

## Leak Check

```bash
niri-leak   # mapping count (healthy: <100, leaky: >1000)
```
