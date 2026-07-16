# K8S INFRASTRUCTURE REFACTORING PLAN (Refined)

## EXECUTIVE SUMMARY

Ground truth as of 2026-05-04:
- 4 nodes: zephyr(32c/31G), nexus(24c/46G), forge(6c/15G+GPU), sentry(16c/31G)
- Cluster runs 6+ agents, mining (28 KH/s), AI inference gateway, monitoring, SSO
- K3s with colmena for NixOS deploys, easykubenix for K8s manifests
- Hardcoded IPs in 50+ files, PVCs pinned to specific nodes, brittle nodeSelectors

This plan fixes fragility incrementally with rollback safety at each step.

---

## PART 1: CENTRALIZED TOPOLOGY MODULE

### File: /etc/nixos/cluster/topology.nix

```nix
{ config, lib, pkgs, ... }:
let
  cluster = {
    name = "j_kro-homelab";
    domain = "cluster.local";
    vip = "10.1.1.100";

    podCidr = "10.244.0.0/16";
    serviceCidr = "10.96.0.0/12";

    hosts = {
      zephyr = {
        ip = "10.1.1.110";
        role = "controlplane";
        cores = 32;
        memory = "31Gi";
        gpu = { type = "RTX3060Ti+RTX3090"; };
      };
      nexus = {
        ip = "10.1.1.120";
        role = "controlplane";
        cores = 24;
        memory = "46Gi";
      };
      forge = {
        ip = "10.1.1.130";
        role = "worker";
        cores = 6;
        memory = "15Gi";
        gpu = { type = "RTX3060Ti"; };
      };
      sentry = {
        ip = "10.1.1.140";
        role = "worker";
        cores = 16;
        memory = "31Gi";
      };
    };

    getHost = name: cluster.hosts.${name};
    getHostIP = name: cluster.hosts.${name}.ip;
    allIPs = lib.mapAttrsToList (_: h: h.ip) cluster.hosts;
  };
in
{
  options.cluster.topology = lib.mkOption {
    type = lib.types.attrs;
    default = cluster;
  };
}
```

### Validation Step

Before any migration, run:

```bash
# Verify topology matches reality
for node in zephyr nexus forge sentry; do
  expected=$(nix eval --raw '.#nixosConfigurations.'$node'.config.cluster.topology.hosts.'$node'.ip')
  actual=$(dig +short $node.cluster.local)
  echo "$node: expected=$expected actual=$actual"
  [ "$expected" = "$actual" ] || echo "  MISMATCH!"
done
```

### /etc/hosts Generation

```nix
networking.extraHosts = lib.concatStringsSep "\n" (
  lib.mapAttrsToList (name: host:
    "${host.ip} ${name}.cluster.local ${name}"
  ) config.cluster.topology.hosts
);
```

---

## PART 2: IP ABSTRACTION & SERVICE DISCOVERY

### A. K8s Internal: Use CoreDNS Service Names

Replace all hardcoded IPs in K8s manifests with service names:
- `10.1.1.120:30888` → `searxng.search.svc.cluster.local:80`
- `10.1.1.110:8040` → `vllm.ai-inference.svc.cluster.local:8040`

### B. NixOS Host-Level: /etc/hosts from topology

Auto-generated from topology.nix (see Part 1). No manual /etc/hosts entries.

### C. SSH Configuration

```nix
programs.ssh.knownHosts = lib.mapAttrs' (name: host:
  lib.nameValuePair name {
    hostNames = [ "${name}.cluster.local" host.ip name ];
    publicKey = builtins.readFile "/etc/ssh/ssh_host_ed25519_key.pub";
  }
) config.cluster.topology.hosts;

programs.ssh.matchBlocks = lib.mapAttrs' (name: host:
  lib.nameValuePair name {
    hostname = host.ip;
    user = "j_kro";
    identityFile = "~/.ssh/id_ed25519";
    controlPath = "~/.ssh/sockets/%r@%h:%p";
    controlMaster = "auto";
    controlPersist = "600";
  }
) config.cluster.topology.hosts;
```

### D. Firewall Rules from CIDRs

```nix
networking.firewall.extraCommands = ''
  iptables -A nixos-fw -s ${cfg.podCidr} -j nixos-fw-accept
  iptables -A nixos-fw -s ${cfg.serviceCidr} -j nixos-fw-accept
'';
```

---

## PART 3: STORAGE CLASS & PVC MIGRATION

### A. Storage Classes

```yaml
# local-fast: WaitForFirstConsumer, Delete reclaim
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-fast
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

---
# nfs-shared: NFS on nexus, Retain
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-shared
provisioner: nfs.csi.k8s.io
volumeBindingMode: Immediate
reclaimPolicy: Retain
parameters:
  server: nexus.cluster.local
  share: /data/shared
```

### B. PVC Migration (Per Workload)

For each StatefulSet pinned to a specific node:

**Step 1: Pre-flight audit**
```bash
# Verify current state
kubectl get pvc -n monitoring -o wide
kubectl get pv -o wide | grep sentry
# Document PVC sizes and storage classes
```

**Step 2: Backup data**
```bash
# Option A: kubectl cp (small data)
kubectl cp monitoring/loki-0:/loki /tmp/loki-backup.tar.gz

# Option B: rsync via host (large data, faster)
ssh sentry "sudo rsync -av /var/lib/rancher/k3s/storage/pvc-XXXX/ /backup/loki-migrate/"
```

**Step 3: Verify backup integrity**
```bash
# CRITICAL: Verify checksums before deleting anything
sha256sum /tmp/loki-backup.tar.gz
# Compare with source
kubectl exec -n monitoring loki-0 -- sha256sum /loki/data-file
```

**Step 4: Migrate**
```bash
# Scale down
kubectl scale statefulset loki -n monitoring --replicas=0

# Delete StatefulSet (orphan PVCs to keep data)
kubectl delete statefulset loki -n monitoring --cascade=orphan

# Delete old PVC
kubectl delete pvc data-loki-0 -n monitoring

# Apply new StatefulSet with local-fast storage class (WaitForFirstConsumer)
kubectl apply -f loki-statefulset-new.yaml

# Wait for new pod
kubectl wait --for=condition=ready pod -l app=loki -n monitoring --timeout=5m

# Restore data
kubectl cp /tmp/loki-backup.tar.gz monitoring/loki-0:/loki/
```

**Step 5: Verify**
```bash
# Check data integrity
kubectl exec -n monitoring loki-0 -- ls -la /loki/
# Check pod is healthy
kubectl describe pod -n monitoring -l app=loki
```

### Migration Order (by risk level)

| Workload | PVC Size | Risk | Node | Priority |
|----------|----------|------|------|----------|
| grafana | ~2GB | Low | sentry | 1 |
| tempo | ~10GB | Low | sentry | 2 |
| loki | ~50GB | Medium | sentry | 3 |
| mimir | ~100GB+ | High | sentry | 4 |

---

## PART 4: SENTRY DISK CLEANUP

### Immediate Actions (non-destructive)

```bash
# 1. Container images
ssh sentry "sudo crictl --runtime-endpoint unix:///run/k3s/containerd/containerd.sock rmi --prune"

# 2. Journal logs (keep 3 days)
ssh sentry "sudo journalctl --vacuum-time=3d"

# 3. Nix GC
ssh sentry "sudo nix-collect-garbage -d"

# 4. Old pod volumes (orphaned)
ssh sentry "sudo find /var/lib/kubelet/pods -type d -mtime +7 -depth 1 -exec rm -rf {} + 2>/dev/null || true"

# 5. K3s containerd snapshots
ssh sentry "sudo crictl --runtime-endpoint unix:///run/k3s/containerd/containerd.sock rmp --all"
```

### Verify available space after cleanup

```bash
ssh sentry "df -h /"
# Target: >50% free on root partition
```

---

## PART 5: STATEFULSET RESCHEDULING

### A. Remove Hardcoded nodeSelectors

Replace:
```yaml
nodeSelector:
  kubernetes.io/hostname: sentry
```

With:
```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values: [sentry, nexus, zephyr]
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: loki
          topologyKey: kubernetes.io/hostname
```

### B. PV Node Affinity Removal (for Retain PVs)

For existing PVs that have node affinity:

```bash
# List PVs with node affinity
kubectl get pv -o json | jq -r '.items[] | select(.spec.nodeAffinity != null) | .metadata.name'

# Patch to remove node affinity (requires PV to be unbound)
# NOTE: This only works after PVC is deleted. Cannot patch bound PV.
# Use the full migration path from Part 3 instead.
```

### C. Important Constraints

1. StatefulSets with `volumeClaimTemplates` create PVCs that bind to the node
   where the pod first schedules. Use `WaitForFirstConsumer` storage class.
2. Existing bound PVCs CANNOT be moved. Must delete and recreate.
3. Never patch PV nodeAffinity on a bound PV - it will not take effect.

---

## PART 6: NETWORK POLICY REFACTORING

### A. Default Deny All

Apply default-deny to each namespace:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
```

### B. Replace IP-Based Rules with Label Selectors

Before:
```yaml
to:
  - ipBlock:
      cidr: 10.1.1.120/32
```

After:
```yaml
to:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: search
    podSelector:
      matchLabels:
        app: searxng
```

### C. Label All Namespaces

```bash
for ns in monitoring auth ai-inference search automation mining; do
  kubectl label namespace $ns kubernetes.io/metadata.name=$ns --overwrite
done
```

### D. Per-Namespace Policies

For each namespace, define:
1. DNS egress (port 53 UDP/TCP to kube-system)
2. Allow ingress from Caddy/ingress controller
3. Allow egress to specific dependent services
4. Allow intra-namespace communication

### E. Test Before Apply

```bash
# Create test namespace with same policies
kubectl create namespace np-test
kubectl apply -f network-policies-test.yaml -n np-test

# Run connectivity test
kubectl run test-src -n np-test --image=busybox --rm -it --restart=Never -- \
  wget -O- --timeout=5 http://searxng.search.svc.cluster.local:8080

# Verify deny works
kubectl run test-deny -n np-test --image=busybox --rm -it --restart=Never -- \
  wget -O- --timeout=5 http://10.1.1.110:8040  # Should fail
```

---

## PART 7: ROLLOUT PLAN

### Phase 1: Topology Module (Week 1, Low Risk)

- [ ] Create `/etc/nixos/cluster/topology.nix`
- [ ] Import in all node configurations
- [ ] Add /etc/hosts generation
- [ ] Add SSH configuration generation
- [ ] Deploy via `colmena switch --on zephyr` (test node first)
- [ ] Verify: `ssh zephyr 'getent hosts nexus.cluster.local'`
- [ ] Deploy to remaining nodes: `colmena switch --on all`
- [ ] Verify: all nodes resolve each other via .cluster.local

### Phase 2: Sentry Disk Cleanup (Week 1, Low Risk)

- [ ] Run cleanup commands from Part 4
- [ ] Verify: `df -h /` shows >50% free on sentry
- [ ] Optional: Migrate monitoring off sentry if still constrained

### Phase 3: Storage Classes (Week 2, Medium Risk)

- [ ] Create `local-fast` and `nfs-shared` StorageClasses
- [ ] Test with dummy workload:
  ```bash
  kubectl run test-sc --image=busybox --restart=Never -- \
    sh -c "echo test > /data/test && cat /data/test"
  # with PVC using local-fast
  ```
- [ ] Verify PVC binds on correct node with WaitForFirstConsumer

### Phase 4: Monitoring Migration (Week 2-3, High Risk)

- [ ] **Pre-flight**: Document all dashboards, alert rules, data retention
- [ ] **Grafana** (~2GB, Low risk):
  - Backup: `kubectl cp monitoring/grafana-0:/var/lib/grafana /backup/grafana`
  - Verify backup checksum
  - Scale to 0, delete STS (orphan), delete PVC
  - Apply new STS with `local-fast` storage class
  - Restore data, verify dashboards load
- [ ] **Tempo** (~10GB, Low risk): Same process as Grafana
- [ ] **Loki** (~50GB, Medium risk):
  - Same process, expect 30-60 min for data copy
  - Verify log queries work after restore
- [ ] **Mimir** (~100GB+, High risk):
  - Consider: do we need all historical metrics?
  - Option: start fresh with empty Mimir, lose old metrics
  - If data needed: rsync via host, not kubectl cp

### Phase 5: Network Policies (Week 3-4, Medium Risk)

- [ ] Label all namespaces
- [ ] Apply default-deny to one namespace (e.g., search)
- [ ] Apply specific allow policies for search
- [ ] Test: verify search services still work
- [ ] Repeat for each namespace
- [ ] Remove old IP-based policies

### Phase 6: Remove Hardcoded IPs (Week 4, Low Risk)

- [ ] `grep -r "10\.1\.1\." /etc/nixos/ --include="*.nix" | grep -v topology.nix`
- [ ] Replace each with reference to `config.cluster.topology.getHostIP "name"`
- [ ] Or use K8s service names where applicable
- [ ] Deploy via colmena, verify all services still functional

---

## ROLLBACK PLAN

Each phase has an independent rollback:

| Phase | Rollback |
|-------|----------|
| Topology | `git revert` + `colmena switch --on all` |
| Disk cleanup | No rollback needed (cleanup only) |
| Storage classes | `kubectl delete sc local-fast nfs-shared` |
| Monitoring | Restore STS from backup YAML + restore PVC data |
| Network policies | `kubectl delete networkpolicy --all -n $namespace` |
| IP removal | `git revert` + `colmena switch --on all` |

### Critical: Always keep backup YAMLs

```bash
# Before each phase
kubectl get all,pvc,pv,configmap,secret,networkpolicy -n monitoring -o yaml > /backup/pre-migration-monitoring.yaml
```

---

## SUCCESS CRITERIA

- [ ] All pods can schedule on any node (no hardcoded nodeSelectors)
- [ ] Storage uses dynamic provisioning (no manual PV management)
- [ ] Network policies use label selectors (no IP blocks)
- [ ] topology.nix is the single source of truth for cluster IPs
- [ ] Zero data loss during migration
- [ ] Monitoring stack operational throughout (alert on downtime)
- [ ] Rollback plan tested for at least Phase 4 (monitoring)

## CONSTRAINTS

- Never use CPU offloading for GPU inference workloads
- Never kill frozen processes (investigate in-place)
- K8s over systemd for all workloads
- All changes deployed via colmena (NixOS) or kubectl (K8s manifests)
- No adaptive sync/VRR anywhere (hardware damage risk)
