# NixOS Config + Infrastructure Deep Scoping Audit
# =================================================
# Comprehensive plan for repository organization, infrastructure mapping,
# security posture, and technical debt remediation.

## TL;DR — Proposed Structure

```
~/Projects/
├── nixos-config/                    # NixOS cluster config ONLY
│   ├── flake.nix
│   ├── modules/                    # Reusable NixOS modules
│   ├── hosts/                      # Per-host configs (zephyr, nexus, etc.)
│   ├── secrets/                    # sops-nix encrypted secrets
│   └── docs/                       # NixOS architecture docs ONLY
│
├── maplespike/                     # MapleSpike product (current repo)
│   ├── packages/                   # Monorepo packages
│   ├── k8s/                        # Kustomize manifests (moved here)
│   ├── config/                     # Grafana dashboards, etc.
│   └── nix/                        # Nix-specific MapleSpike config
│
├── homelab-ops/                    # NEW: Operational runbooks + automation
│   ├── scripts/                    # Cloudflare, switches, backups
│   ├── runbooks/                   # docs/operations/*.md
│   └── infrastructure/             # Network diagrams, inventory
│
├── hermes-skills/                  # NEW: Hermes agent skills
│   ├── devops/
│   ├── security/
│   ├── kubernetes/
│   └── software-development/
│
└── infrastructure-docs/            # NEW: Architecture + security docs
    ├── architecture.md
    ├── security-posture.md
    └── infrastructure-inventory.md
```

---

## Phase 1: Repository Scoping

### What BELONGS in nixos-config

**Core NixOS infrastructure:**
- ✅ `modules/` — Reusable NixOS modules (network, services, security, etc.)
- ✅ `hosts/` — Per-host configurations (zephyr, nexus, forge, sentry, krash3)
- ✅ `secrets/` — sops-nix encrypted secrets (structured by service)
- ✅ `flake.nix` — Nix flake definition + inputs
- ✅ `scripts/nixos-rebuild-safe.sh` — NixOS rebuild wrapper
- ✅ `scripts/deploy.sh` — Colmena deployment script
- ✅ `docs/ARCHIVE/` — Historical NixOS docs (keep)
- ✅ `docs/LIVE/` — Live infrastructure docs (keep)

**K3s cluster configuration (Nix-managed):**
- ✅ `kubernetes/` — Nix modules for k3s configuration (agent config, version, node labels)
- ✅ `kubernetes-manifests/` — Static K8s manifests (cluster-wide RBAC, storage-classes, security-baseline)
  - These are INFRASTRUCTURE manifests (network-policies, security-baseline, storage-classes)
  - NOT application manifests (those go in maplespike/k8s/)

**Build/CI infrastructure:**
- ✅ `scripts/ci/` — CI/CD scripts for nixos-config
- ✅ `.github/workflows/` — GitHub Actions for nixos-config repo

**Hardware control:**
- ✅ `modules/hardware/` — GPU passthrough, RGB control, hardware monitoring
- ✅ `scripts/gpu-profiles/` — GPU performance profiles (mining vs AI)
- ✅ `plasmoids/` — KDE plasmoids for system monitoring

### What DOES NOT BELONG in nixos-config

**Move to homelab-ops/ (NEW):**
- ❌ `scripts/CLOUDFLARE_SECURITY_FIXES.md/sh` — Cloudflare runbooks
- ❌ `scripts/switch-*` — TP-Link switch management scripts
- ❌ `scripts/backup-*` — Backup scripts (garage, switches)
- ❌ `scripts/cleanup-*` — Cleanup scripts (zombie pods, stale locks)
- ❌ `scripts/cluster-watchdog.sh` — Cluster monitoring daemon
- ❌ `scripts/monitor-sensors.sh` — Hardware monitoring
- ❌ `scripts/mining-*` — Mining profitability/scheduler
- ❌ `scripts/tplink-*` — TP-Link automation
- ❌ `scripts/auto-update-*` — Auto-update scripts
- ❌ `scripts/secret-rotation.sh` — Secret rotation schedule
- ❌ `scripts/preflight-check.sh` — Pre-deployment checks (ops checklist)
- ❌ `scripts/hsync.sh` — Homelab sync tool
- ❌ `scripts/test-*` — Integration tests (move to homelab-ops/tests/)
- ❌ `scripts/archive/cloudflare-*` — Old Cloudflare scripts
- ❌ `docs/SERVICE-AUDIT-FIXES.md` — Ops history
- ❌ `docs/SERVICE-FIXES-COMPLETED.md` — Ops history

**Move to hermes-skills/ (NEW):**
- ❌ `skills/` — Hermes agent skills (40+ skills)
- ❌ `.hermes/skills/` — Hermes skills backup

**Move to maplespike/ (application):**
- ❌ `k8s/` — Duplicate of maplespike k8s/ directory (delete)
- ❌ `k8s/examples/` — Example manifests (move to maplespike/docs/examples/)
- ❌ `k8s/opencode/` — Opencode-specific manifests (move to maplespike/k8s/overlays/)

**Move to hermes-agent repo:**
- ❌ `.claude/skills/` — Claude skills (move to Hermes repo)

**Delete/Clean:**
- ❌ `$out` — Nix build artifacts (gitignored)
- ❌ `.gcroots` — Nix garbage collection roots (gitignored)
- ❌ `.direnv` — Direnv cache (gitignored)
- ❌ `.pnpm-store` — pnpm cache (gitignored)
- ❌ `.pytest_cache` — pytest cache (gitignored)
- ❌ `.ruff_cache` — ruff cache (gitignored)
- ❌ `.sisyphus` — Sisyphus cache (gitignored)
- ❌ `.understand-anything` — AI tool cache (gitignored)
- ❌ `Cache/` — Build cache (gitignored)
- ❌ `htmlcov/` — Test coverage reports (gitignored)
- ❌ `tmp/` — Temporary files (gitignored)
- ❌ `containers/` — Container build artifacts (gitignored)
- ❌ `manifests/` — Unclear purpose (review, likely delete)

### Repository Migration Plan

**Step 1: Create new repos**
```bash
cd ~/Projects
mkdir homelab-ops hermes-skills infrastructure-docs
cd homelab-ops && git init
cd ../hermes-skills && git init
cd ../infrastructure-docs && git init
```

**Step 2: Move scripts to homelab-ops**
```bash
cd /etc/nixos

# Move ops scripts
scripts/CLOUDFLARE_SECURITY_FIXES.md ~/Projects/homelab-ops/runbooks/
scripts/cloudflare-security-fixes.sh ~/Projects/homelab-ops/scripts/

# Move all switch scripts
scripts/tplink* ~/Projects/homelab-ops/scripts/
scripts/switch-* ~/Projects/homelab-ops/scripts/

# Move backup scripts
scripts/backup-* ~/Projects/homelab-ops/scripts/

# Move monitoring scripts
scripts/monitor-sensors.sh ~/Projects/homelab-ops/scripts/
scripts/mining-* ~/Projects/homelab-ops/scripts/

# Move cleanup scripts
scripts/cleanup-* ~/Projects/homelab-ops/scripts/
scripts/cluster-watchdog.sh ~/Projects/homelab-ops/scripts/

# Move ops docs
docs/SERVICE-*.md ~/Projects/homelab-ops/runbooks/
```

**Step 3: Move skills to hermes-skills**
```bash
cd /etc/nixos

# Move Hermes skills
cp -r skills/* ~/Projects/hermes-skills/
cp -r .hermes/skills/* ~/Projects/hermes-skills/
cp -r .claude/skills/* ~/Projects/hermes-skills/

# Move Claude skills
rm -rf .claude/skills
```

**Step 4: Clean up nixos-config**
```bash
cd /etc/nixos

# Remove k8s duplicates (maplespike has these)
rm -rf k8s/

# Remove k8s examples (move to maplespike)
mv k8s/examples/* ~/Projects/maplespike/docs/examples/
rm -rf k8s/examples/
rm -rf k8s/opencode/

# Clean build artifacts
rm -rf $out .gcroots .direnv .pnpm-store .pytest_cache .ruff_cache
rm -rf .sisyphus .understand-anything Cache htmlcov tmp containers

# Remove unclear directory
rm -rf manifests/

# Move archive scripts to homelab-ops
mv scripts/archive/* ~/Projects/homelab-ops/scripts/archive/
rmdir scripts/archive
```

---

## Phase 2: Infrastructure Scoping

### Services Running Across the Cluster

**By Category:**

| Category | Service | Primary Host | Redundancy | Notes |
|----------|---------|--------------|------------|-------|
| **Kubernetes** | k3s control plane | nexus, forge, sentry | HA (3 nodes) | etcd cluster, load-balanced API (10.1.1.100 VIP) |
| | k3s agent | zephyr, krash3 (planned) | Multiple | zephyr runs worker pods |
| **Storage** | Garage S3 | forge | Multi-zone | Distributed object storage |
| | NFS | nexus | Single | Network-attached storage |
| **Monitoring** | Prometheus | nexus | — | Scrapes cluster metrics |
| | Grafana | nexus | — | Dashboard visualization |
| | Alertmanager | nexus | — | Alert routing |
| **AI Inference** | vLLM (Gemma-4) | forge | — | AMD GPU inference |
| | llama.cpp (Zephyr) | zephyr | — | CPU fallback inference |
| | llama.cpp (Sentry) | sentry | — | CPU fallback inference |
| **Search** | SearXNG | k3s (nexus/forge/sentry) | Replicated | Distributed search |
| **Identity** | Casdoor | k3s (nexus) | — | OIDC/SAML auth |
| **Workflow** | n8n | k3s (zephyr) | — | Automation workflows |
| **Network** | Caddy | nexus | — | Reverse proxy, Let's Encrypt |
| | Keepalived | All nodes | VIP failover | 10.1.1.100 VIP management |
| | Unbound DNS | All nodes | Any | Local DNS forwarder (.lan domains) |
| **Mining** | PeakMiner | zephyr, krash1.5 | Multiple | NVIDIA GPU mining |
| | xmrig-proxy | krash3, krash1.5 | — | Stratum proxy |
| **Database** | PostgreSQL (frostbite) | k3s | — | Frostbite Gazette DB |
| **Media** | ComfyUI | zephyr | — | Image generation |

### Overlaps/Redundancies

**Duplicate Services:**
1. **DNS:** Unbound runs on ALL nodes for .lan domains (good redundancy)
   - No action needed — this is intentional

2. **SearXNG:** Multiple instances on k3s (nexus/forge/sentry)
   - Intentional for load balancing
   - No action needed

3. **AI Inference:** Multiple llama.cpp instances (zephyr, sentry)
   - Intentional fallback chain
   - No action needed

4. **K3s manifests:** `k8s/`, `kubernetes-manifests/` in nixos-config vs `k8s/` in maplespike
   - CONFUSION — need to consolidate
   - Fix: Keep infrastructure manifests in nixos-config, app manifests in maplespike

**Unclear Boundaries:**
- `kubernetes-manifests/` in nixos-config has both infra and app manifests
- `k8s/` in maplespike has app manifests (correct)
- Fix: Move only infra manifests to nixos-config, app manifests stay in maplespike

### Missing Pieces

**Infrastructure:**
1. **Centralized logging** (ELK/Loki) — ❌ Missing
2. **Backup automation** (manual scripts exist, no automated solution)
3. **Secret rotation** (schedule documented, no automation)
4. **Disaster recovery** (no documented DR plan)
5. **Capacity planning** (no monitoring of resource utilization)
6. **Cost tracking** (no cost optimization or cloud spend monitoring)

**Security:**
1. **Network segmentation** (VLANs exist, no enforcement)
2. **Zero Trust network** (Tailscale exists, no cluster-wide rollout)
3. **Vulnerability scanning** (no regular scanning)
4. **Compliance auditing** (no audit trail)
5. **Incident response** (no documented IR plan)

**Observability:**
1. **Distributed tracing** (no Jaeger/Tempo)
2. **Application performance monitoring** (no APM)
3. **Real user monitoring** (no RUM)

---

## Phase 3: Security Scoping

### Secrets Management

**Current State:**
- ✅ sops-nix for NixOS secrets (`/etc/nixos/secrets/`)
- ✅ Git-crypt for some repos (legacy, migrating to sops-nix)
- ✅ SealedSecrets for K8s (managed by k3s-sealed-secrets-controller)
- ❌ No central secret rotation (manual script exists)
- ❌ No secret versioning (no audit trail)
- ❌ No secret expiration enforcement

**Secret Inventory:**

| Category | Secrets | Location | Rotation Schedule |
|----------|---------|----------|-------------------|
| AI Services | gemini-api-key, nvidia-api-key, zai-api-key, xai-access-token | `secrets/ai/` | quarterly |
| CI/CD | github-token, github-runner-pat, npm-token, gitea-runner-* | `secrets/ci/` | quarterly |
| Cloud | cloudflare-api-token, cloudflare-global-api-key, cloudflared-token, tailscale-* | `secrets/cloud/` | quarterly |
| Infrastructure | initrd-ssh-* hosts, ssh-ca-key, switch-admin | `secrets/infra/` | yearly |
| K8s | k3s-cluster-token, ai-gateway-zai, casdoor-hermes-jwt | `secrets/k8s/` | quarterly |
| Mining | xmrig-* tokens | `secrets/mining/` | quarterly |
| Monitoring | grafana-admin-* | `secrets/monitoring/` | yearly |
| Self-hosting | nextcloud-admin, vaultwarden-* | `secrets/selfhosting/` | yearly |
| Storage | garage-* keys | `secrets/storage/` | yearly |

**Age Key Distribution:**
- ✅ zephyr key (age1p98yp8w64rdugp03332gxnz5q2vcnucn69cs5qm6s2l2u7epqfcqmu2pqe)
- ✅ krash3 key (age1ehx7aedejaf3tqje43h4nj33c3s4nctet6mpys9pyegz4wexa3jqhgg0dz)
- ❌ No YubiKey backup (yubikey-nano-mgmt-key.age exists, not configured)

### Exposure Analysis

**External Exposure:**
1. **Cloudflare** (zone: 9062487114ef5404de8de6689cb54895)
   - Domains: reverb256.ca
   - WAF: Enabled (Turnstile, AI Labyrinth not configured)
   - DNS: Some unproxied A records (exposes origin IPs)
   - Status: ⚠️ 46 security insights (DMARC, security.txt, AI bots)

2. **Tailscale** (cluster mesh)
   - Nodes: zephyr, nexus, forge, sentry
   - ACLs: Not documented
   - Status: ⚠️ Unknown ACL configuration

3. **Public endpoints:**
   - maplespike.lan (Caddy TLS, internal only)
   - searxng.lan (Caddy TLS, internal only)
   - Status: ✅ No direct internet exposure (VLANs, NAT)

**Internal Exposure:**
1. **Lack of network segmentation**
   - All hosts on same VLAN (10.1.1.0/24)
   - No intra-cluster firewall rules
   - Status: ⚠️ Lateral movement possible

2. **Unbound DNS**
   - All nodes query local unbound
   - No DNS filtering for malicious domains
   - Status: ⚠️ No DNS security layer

3. **SSH access**
   - SSH CA for host authentication ✅
   - Agent signing for host key verification ✅
   - No MFA required for SSH
   - Status: ⚠️ Passwordless auth, no MFA

### Access Controls

**Current State:**
| Resource | Auth Method | MFA | RBAC | Audit |
|----------|-------------|-----|------|-------|
| K8s API | Casdoor OIDC | ✅ (Casdoor) | ✅ (RBAC) | ❌ |
| K8s Nodes | SSH (agent) | ❌ | ❌ | ❌ |
| Web Apps | Casdoor OIDC | ✅ (Casdoor) | ✅ | ❌ |
| NixOS hosts | SSH (agent) | ❌ | ❌ | ❌ |
| Garage S3 | Static API key | ❌ | ❌ | ❌ |
| Grafana | Static password | ❌ | ✅ (RBAC) | ❌ |

**Recommendations:**
1. Enable MFA for SSH (hardware key or TOTP)
2. Implement SSH RBAC (sudo, role-based access)
3. Enable audit logging for SSH
4. Rotate API keys regularly
5. Implement session timeout for web apps
6. Enable audit logging for K8s (auditd)

---

## Phase 4: NixOS Configuration Scoping

### Modules vs Host Configs

**Current Structure:**
```
modules/
├── common/              # Shared config (networking, firewall)
├── desktop/             # Desktop environment (Niri, Wayland)
├── development/         # Dev tools (git, editors, languages)
├── gaming/              # Gaming setup (Steam, Lutris)
├── hardware/            # Hardware-specific (GPU, RGB)
├── home-manager/        # User-level config
├── lib/                 # Nix library functions
├── multimedia/          # Media apps (VLC, MPV)
├── network/             # Networking modules
├── profiles/            # Host profiles (workstation, server)
├── scripts/             # Helper scripts
├── security/            # Security hardening
├── services/            # Service modules (k3s, monitoring)
├── shell/               # Shell config (fish, zsh)
└── system/              # System-level config (boot, kernel)

hosts/
├── zephyr/              # Main workstation
├── nexus/               # K8s control plane #1
├── forge/               # K8s control plane #2
├── sentry/              # K8s control plane #3
└── krash3/              # K8s agent + Windows VM host
```

**Analysis:**
- ✅ Good separation of concerns
- ✅ Modules are reusable across hosts
- ✅ Host configs are minimal (import modules)
- ⚠️ Some modules are overly specific (gaming, multimedia)
- ⚠️ Some modules duplicate functionality (network vs networking)

### Reusable vs Specific

**Reusable Modules (good):**
- `modules/network/cluster-dns.nix` — Unbound DNS config
- `modules/services/k3s-cluster.nix` — K3s agent config
- `modules/security/casdoor-hermes.nix` — Casdoor auth
- `modules/system/distributed-builds.nix` — Remote builds

**Host-Specific Modules (OK):**
- `modules/hardware/gpu-passthrough.nix` — GPU passthrough (krash3 only)
- `modules/desktop/niri-desktop.nix` — Niri desktop (zephyr only)
- `modules/gaming/steam-gaming.nix` — Steam setup (zephyr only)

**Modules to Consolidate:**
1. `modules/network/` + `modules/networking/` → `modules/network/`
2. `modules/profiles/` (overlaps with host configs)

### Technical Debt

**Identified Issues:**
1. **Duplicate k8s directories** (k8s/, kubernetes/, kubernetes-manifests/)
   - Fix: Consolidate to `kubernetes-manifests/` (infra only)

2. **Scripts in root** (80+ scripts in `scripts/`)
   - Fix: Move to homelab-ops/ repository

3. **Mixed docs** (AGENTS.md, SERVICE-AUDIT-FIXES.md)
   - Fix: Move ops docs to homelab-ops/

4. **Skills in nixos-config** (40+ skills in `skills/`)
   - Fix: Move to hermes-skills/ repository

5. **No module documentation**
   - Fix: Add README.md to each module directory

6. **Secrets not centrally managed**
   - Fix: Centralize secret rotation in homelab-ops/

7. **No automated testing**
   - Fix: Add NixOS VM tests

---

## Phase 5: Best Practices Research

### Repository Organization Best Practices

**Industry Standards:**
1. **Monorepo vs Polyrepo**
   - Monorepo: Single repo for all related services (Google, Meta)
   - Polyrepo: Separate repo per service (Amazon, Netflix)
   - Hybrid: Infra monorepo + app monorepos (recommended for homelab)

2. **Infrastructure as Code (IaC) Repository Structure**
   ```
   infra/
   ├── terraform/          # Cloud infrastructure
   ├── ansible/            # Configuration management
   ├── packer/             # Image building
   ├── kubernetes/         # K8s manifests
   ├── scripts/            # Helper scripts
   └── docs/               # Infrastructure docs
   ```

3. **Application Monorepo Structure**
   ```
   app/
   ├── packages/           # Monorepo packages
   ├── apps/               # Applications
   ├── services/           # Microservices
   ├── docs/               # App docs
   └── k8s/                # K8s manifests
   ```

4. **Runbook Repository Structure**
   ```
   runbooks/
   ├── scripts/            # Automation scripts
   ├── docs/               # Operational docs
   ├── playbooks/          # Ansible playbooks
   └── diagrams/           # Network diagrams
   ```

### NixOS Configuration Best Practices

**Official NixOS Recommendations:**
1. **Use flakes** for reproducible builds
2. **Modularize configuration** (reusable modules)
3. **Document host-specific quirks** in host config files
4. **Use sops-nix** for secrets (not git-crypt)
5. **Keep secrets in `secrets/` directory** with sops-nix
6. **Test configuration** with `nixos-rebuild build` before `switch`
7. **Use Colmena** for multi-host deployments
8. **Keep `flake.lock`** for reproducibility

**Community Best Practices:**
1. **Separate concerns:**
   - `modules/common/` — shared config
   - `modules/services/` — service modules
   - `modules/system/` — system-level config
   - `hosts/` — per-host configs

2. **Document modules:**
   - Add `README.md` to each module
   - Describe purpose, options, and usage

3. **Use descriptive module names:**
   - ✅ `modules/services/k3s-cluster.nix`
   - ❌ `modules/services/kubernetes.nix` (ambiguous)

4. **Test configuration:**
   - Use NixOS VM tests
   - Test on a staging host before production

### Security Best Practices

**Secrets Management:**
1. **Never commit secrets** to version control
2. **Use sops-nix** for NixOS secrets
3. **Rotate secrets regularly** (quarterly for API keys, yearly for long-term)
4. **Use hardware keys** (YubiKey) for critical secrets
5. **Audit secret access** (who accessed what, when)

**Network Security:**
1. **Segment network** (VLANs for different zones)
2. **Firewall by default deny** (allow only necessary traffic)
3. **Use VPN for remote access** (Tailscale, WireGuard)
4. **Enable IDS/IPS** (Snort, Suricata)
5. **Monitor network traffic** (netflow, Zeek)

**Identity and Access:**
1. **Use MFA everywhere** (hardware key preferred)
2. **Implement RBAC** (role-based access control)
3. **Audit access** (who accessed what, when)
4. **Use short-lived credentials** (no permanent API keys)
5. **Implement session timeout** (auto-lock after inactivity)

**Observability:**
1. **Centralized logging** (ELK, Loki)
2. **Distributed tracing** (Jaeger, Tempo)
3. **Metrics everywhere** (Prometheus, Grafana)
4. **Alert on anomalies** (Alertmanager, PagerDuty)
5. **Document incident response** (runbooks, playbooks)

---

## Execution Plan

### Week 1: Repository Migration

**Day 1-2: Create new repos**
```bash
cd ~/Projects
git init homelab-ops hermes-skills infrastructure-docs

# Add .gitignore files
cat > homelab-ops/.gitignore << 'EOF'
# Logs
*.log
# Temp files
*.tmp
# Secrets (encrypted only)
!*.age
!*.yaml (sops)
EOF
```

**Day 3-4: Move scripts to homelab-ops**
```bash
# Move ops scripts (see Phase 1, Step 2 for detailed commands)
# Commit and push to GitHub
cd ~/Projects/homelab-ops
git add .
git commit -m "Initial commit: ops scripts from nixos-config"
git remote add origin git@github.com:reverb256/homelab-ops.git
git push -u origin main
```

**Day 5: Move skills to hermes-skills**
```bash
# Move skills (see Phase 1, Step 3 for detailed commands)
# Commit and push
cd ~/Projects/hermes-skills
git add .
git commit -m "Initial commit: Hermes skills from nixos-config"
git remote add origin git@github.com:reverb256/hermes-skills.git
git push -u origin main
```

### Week 2: Clean up nixos-config

**Day 1-2: Remove k8s duplicates**
```bash
cd /etc/nixos
rm -rf k8s/
# Commit changes
git add -A
git commit -m "Remove duplicate k8s/ directory (maplespike has these)"
git push
```

**Day 3-4: Clean build artifacts**
```bash
# Update .gitignore
cat >> .gitignore << 'EOF'
# Nix build artifacts
$result
.gcroots
.direnv
.pnpm-store
.pytest_cache
.ruff_cache
.sisyphus
.understand-anything
Cache
htmlcov
tmp
containers
EOF

# Clean directories
rm -rf $out .gcroots .direnv .pnpm-store .pytest_cache .ruff_cache
rm -rf .sisyphus .understand-anything Cache htmlcov tmp containers
rm -rf manifests/

# Commit
git add .gitignore
git commit -m "Add comprehensive .gitignore and clean build artifacts"
git push
```

**Day 5: Add module documentation**
```bash
# Generate README.md for each module
# Use AI or manual effort to document module purpose
```

### Week 3: Security Hardening

**Day 1-2: Fix Cloudflare security issues**
```bash
# Follow CLOUDFLARE_SECURITY_FIXES.md
# 1. Add DMARC TXT record
# 2. Enable proxy on unproxied A records
# 3. Delete dangling A records
# 4. Deploy security.txt
# 5. Enable AI Labyrinth
# 6. Create WAF rule for AI bots
```

**Day 3-4: Configure YubiKey backup**
```bash
# Configure YubiKey as backup age key
# Follow https://github.com/FiloSottile/age#yubikey
```

**Day 5: Enable MFA for SSH**
```bash
# Configure YubiKey or TOTP for SSH MFA
# Add to nixos-config hosts/*/configuration.nix
```

### Week 4: Documentation

**Day 1-2: Create infrastructure docs**
```bash
cd ~/Projects/infrastructure-docs
cat > architecture.md << 'EOF'
# NixOS Cluster Architecture

## Overview
6-node NixOS homelab cluster running k3s, GPU mining, AI inference, and self-hosted services.

## Network Topology
...

## Services
...

## Security
...
EOF
```

**Day 3-4: Create security posture doc**
```bash
cat > security-posture.md << 'EOF'
# Security Posture

## Threat Model
...

## Attack Surface
...

## Mitigations
...

## Incident Response
...
EOF
```

**Day 5: Create infrastructure inventory**
```bash
cat > infrastructure-inventory.md << 'EOF'
# Infrastructure Inventory

## Hardware
...

## Software
...

## Services
...

## Dependencies
...
EOF
```

---

## Verification Checklist

### Repository Scoping
- [ ] nixos-config contains only NixOS config
- [ ] homelab-ops created and populated with ops scripts
- [ ] hermes-skills created and populated with skills
- [ ] maplespike contains app manifests (k8s/)
- [ ] No duplicate k8s directories

### Infrastructure Scoping
- [ ] All services documented in inventory
- [ ] Redundancies identified and intentional
- [ ] Missing pieces prioritized
- [ ] No unclear boundaries

### Security Scoping
- [ ] All secrets identified and rotation scheduled
- [ ] External exposure analyzed
- [ ] Access controls documented
- [ ] MFA enabled for critical services

### NixOS Configuration Scoping
- [ ] Modules documented
- [ ] Host configs minimal
- [ ] Technical debt tracked
- [ ] No dead code

---

## Next Steps

1. **Execute migration plan** (4 weeks)
2. **Update AGENTS.md** in each repo
3. **Integrate ops scripts** with Hermes skills
4. **Set up automated secret rotation**
5. **Implement security hardening**
6. **Document cluster architecture**
7. **Add observability (logging, tracing)**
8. **Implement disaster recovery plan**
---

> Snapshot from August 2026 cleanup; verify current state via `/etc/nixos/SOPS-NIX.md`.

## See Also — SOPS-NIX (canonical on this host)

For canonical sops-nix status, key file location (`/etc/nixos/.age/key.txt`),
registry module structure (`/etc/nixos/modules/system/sops-secrets-registry.nix`),
current recipients (`/etc/nixos/.sops.yaml`), and recovery workflow, see
`/etc/nixos/SOPS-NIX.md`.

Quick facts that hold on this NixOS host (zephyr):
- Registry `services.sops-secrets-registry.enable` defaults to `false` on
  all 5 hosts (forge, nexus, sentry, zephyr, krash3); the registry's
  `mkIf` block is currently inert and `config.sops.secrets` evaluates to
  `[]` until a host opts in.
- 0/135 existing encrypted files decrypt locally today (legacy
  recipients pre-date the single-pubkey `.sops.yaml` policy). The
  `/etc/nixos/.sops.yaml` already names the local pubkey, so new
  encryptions will decrypt on zephyr.
- After any `age-keygen` / `sops updatekeys` operation, sync the user
  key to the canonical location:
  `sudo cp ~/.age/key.txt /etc/nixos/.age/key.txt && sudo chown root:root /etc/nixos/.age/key.txt && sudo chmod 600 /etc/nixos/.age/key.txt`.
