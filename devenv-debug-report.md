# Devenv/NixOS Integration Issue Debug Report

## Problem Summary
During `nixos-rebuild switch --flake /etc/nixos#zephyr --upgrade-all`, the build failed with repeated errors:
```
Changes that will be made to bash/devenv.nix:
 { pkgs, lib, config, inputs, ... }:
 {
   # https://devenv.sh/basics/
   env.GREET = "devenv";
   ...
 }
Error:   × IO error: not a terminal
```

Similar errors appeared for `fish/devenv.nix`, followed by package build timeouts and eventual user interruption.

## Root Cause Analysis

### 1. System Configuration
- **direnv enabled**: Found in `/etc/nixos/modules/development/tools.nix` (lines 2-3)
  ```nix
  programs.direnv.enable = true;
  programs.direnv.nix-direnv.enable = true;
  ```
- **devenv package installed**: Same file (line 74)
  ```nix
  environment.systemPackages = with pkgs; [
    # ... other packages
    devenv
  ];
  ```
- **No devenv NixOS module**: Zero references to `programs.devenv.enable` or similar configuration

### 2. Problematic .envrc File
Located at: `/etc/nixos/.pi/npm/node_modules/@ifi/oh-pi-skills/skills/rust-workspace-bootstrap/template/.envrc`
Content:
```bash
export DIRENV_WARN_TIMEOUT=20s
eval "$(devenv direnvrc)"
use devenv -c
```

### 3. Failure Mechanism
1. During `nixos-rebuild switch`, direnv activates (enabled via NixOS)
2. direnv encounters the .envrc file in the template directory
3. direnv executes: `eval "$(devenv direnvrc)"`
4. devenv initializes and attempts to generate shell integration files:
   - `bash/devenv.nix` 
   - `fish/devenv.nix`
5. Since no devenv configuration exists (no devenv.yaml/devenv.nix), devenv fails
6. Failure manifests as "IO error: not a terminal" because devenv tries to write files in non-interactive build context

### 4. Evidence
- devenv version 2.0.6 confirmed installed
- No `bash/devenv.nix` or `fish/devenv.nix` files exist anywhere in system
- devenv showed what it would write (the configuration block) before failing to write it

## Solutions

### Option 1: Remove Problematic .envrc (Recommended)
```bash
rm /etc/nixos/.pi/npm/node_modules/@ifi/oh-pi-skills/skills/rust-workspace-bootstrap/template/.envrc
```
**Rationale**: The .envrc is in a template directory under skills - likely not needed for actual system operation. This prevents the devenv initialization trigger during builds.

### Option 2: Create Proper Devenv Configuration
Create a `devenv.yaml` or `devenv.nix` in the same directory as the .envrc file with appropriate configuration.

### Option 3: Disable Devenv System Package
Remove `devenv` from `environment.systemPackages` in `/etc/nixos/modules/development/tools.nix` if devenv is not used system-wide.

### Option 4: Temporary Direnv Disable During Rebuilds
```bash
# In modules/development/tools.nix:
programs.direnv.enable = false;
programs.direnv.nix-direnv.enable = false;
```

## Key Technical Insights

1. **Devenv/Direnv Interaction**: direnv-triggered devenv initialization attempts to create shell-specific configuration files
2. **Configuration Requirement**: devenv requires devenv.yaml or devenv.nix to function properly
3. **Context Sensitivity**: devenv file operations fail in non-interactive contexts (like nixos-rebuild)
4. **Template Artifact Risk**: Configuration files in template directories can interfere with system operations when discovered by tools like direnv
5. **Toolchain Separation**: Development environment tools (direnv/devenv) should be carefully isolated from system management operations

## Preventive Measures

1. **Template Directory Management**: Add `.direnvrc` with `direnv deny` to template directories to prevent accidental activation
2. **Configuration Validation**: Verify that any development tool configurations don't interfere with system operations
3. **Selective Tool Installation**: Consider installing development tools like devenv only in user environments, not system-wide via NixOS
4. **Build Context Awareness**: Use tools that detect non-interactive contexts and adjust behavior accordingly

## Resolution Recommendation
Given that:
- The problematic .envrc is in a template directory under skills
- No actual devenv configuration exists system-wide
- devenv is installed but not actively configured for use
- The error occurs during system rebuild operations

**Remove the .envrc file** is the safest immediate solution. This eliminates the trigger while maintaining system integrity and preserving the ability to use devenv in appropriate development contexts where properly configured.

## Memory Entry
Saved to persistent memory for future reference:
"User j_kro encountered nixos-rebuild switch failures due to devenv trying to generate shell integration files (bash/devenv.nix, fish/devenv.nix) in a non-interactive context. Root cause: direnv enabled + .envrc file in template directory triggering devenv initialization without proper devenv configuration. Solution: remove the problematic .envrc file or create proper devenv configuration."

---
Report generated: $(date)
System: Zephyr (NixOS/K3s homelab node)