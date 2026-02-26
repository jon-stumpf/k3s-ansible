# Commit Analysis

## Commit Groups

### 1. Ansible Lint / Code Quality
Fixes across the entire codebase to satisfy `ansible-lint` rules:
- FQCN module names (`ansible.builtin.*`, `ansible.posix.*`, `community.general.*`)
- Truthy values (`yes/no` → `true/false`)
- `no-changed-when` errors
- `name-uppercase` errors
- Task ordering warnings
- `shell` → `command` replacements
- Jinja template placement in `name:` fields
- Octal modes → symbolic modes
- Role renaming to avoid path-based names (`ha_kube-vip` → `ha_kube_vip`)
- Suppressed yaml-lint line-length warnings

---

### 2. Upstream Naming Alignment
Renaming to match the current `k3s-io/main` conventions:
- Host groups: `master/node` → `k3s_server/k3s_agent` → `server/agent`
- Roles: `k3s/master`, `k3s/node` → `k3s/server`, `k3s/agent` → `k3s_server`, `k3s_agent`
- Service: `k3s-node` → `k3s-agent`
- Variables: `master_ip` → `server_ip` → `apiserver_endpoint` → `api_endpoint`
- Args: `install_k3s_server_args`, `install_k3s_agent_args`
- Added `ansible.builtin.*` FQCN prefixes throughout
- Added `cluster_port` / `api_port`
- Added `playbook/reboot.yml`
- Added `collections/requirements.yml`, `Vagrantfile`, `.ansible-lint`
- Converted inventory from INI to YAML format

---

### 3. HA Cluster Methods
Full implementation of multiple HA VIP methods as roles:
- **kube-vip**: role-based implementation, updated to v0.6.3, added load-balancer capability, cleanup on install/reset
- **keepalived**: role-based implementation with dictionary-based variable cleanup
- **etcd (embedded)**: converted `k3s-init.yml` to a role (`ha/etcd`), `ha_enabled` flag
- **Externally-provided VIP**: separate cluster method option
- Default HA method set to kube-vip
- Fixed HA method keepalived
- Configuration checks for `ha_cluster_vip` and `ha_cluster_method`

---

### 4. Configuration Validation Role
- Added `roles/config-check` to validate required variables
- Enforced `k3s_token` must be defined
- Added checks for HA-specific variables
- Handled empty `group_vars/all.yml` gracefully

---

### 5. group_vars / Inventory Simplification
Cleaned up and restructured configuration variables:
- Moved `cluster_config` to `inventory/sample/group_vars/all.yml`
- Simplified `all.yml` using dictionaries for related variables
- Eliminated `cluster-config` role (inlined)
- Added `inventory/sample/group_vars/README.md`
- Made `cluster_config` and `ha_cluster_method` optional with defaults
- Renamed `extra_*` variables to `k3s_*`

---

### 6. Reset Role Improvements
- Brought reset in line with `k3s-uninstall.sh` behavior
- Removed cluster VIP from interface during reset
- Renamed `keep_binaries` → `remove_packages`; applied to `k3s-selinux` package management
- Removed `~/.kube/config` and `~/.kube` directory on reset
- Fixed `pkill` to use `k3s_server_location`
- Fixed symlinked command errors on reset when data dir is non-default

---

### 7. Download Role Enhancements
- Refactored `roles/download/tasks/main.yml` using blocks
- Added support for version channels (e.g., `stable`)
- Added commit-based installation (`k3s_commit`)
- Configurable bin directory; added `ctr` to symlink list
- Moved `vars` to `defaults`

---

### 8. OS / Platform Support
- **Raspberry Pi / Debian**: fixed prereq tasks, Bookworm iptables install, ip-netns bug mitigation
- **ArchLinux (ARM64)**: added support
- **RedHat / RHEL 8.4**: added support
- Reduced differences between OS-specific prereq task files
- Only forward IPv6 when IPv6 interfaces exist
- Only reload k3s service when service files change

---

### 9. Additional Features
- Added ability to install additional manifests and packages for servers and agents
- Added optional config file support
- Added AppArmor checks
- Added firewall exception support
- Added dependent Ubuntu packages
- Checked for broken iptables
- Added variables for default directory locations and symlink to alternate data directory
- Installed Ubuntu Raspi extra packages

---

### 10. Kubeconfig Management
- Saved kubeconfig to localhost as `playbook/cluster.conf`
- Saved both `localhost` and `{{ api_endpoint }}` versions of kubeconfig on each server

---

### 11. Idempotency / Control Flow
- `site.yml` made reentrant (only reports changes when appropriate)
- Wait for control plane before configuring agents (increased to 60s)
- Prevented unnecessary tasks causing state changes
- Fixed tasks that reported changes when there were none
- Only reload k3s service when service file(s) change
- Wait for control plane only happens once
- Updated register variables to avoid potential conflicts

---

### 12. k3s Service Memory Variables
- Added memory-related variables for the k3s service

---

### 13. Documentation
- Updated and expanded `README.md`
- Added `TODO.md` with progress tracking
- Updated documentation links to be more specific
- Added Turing Pi HA inventory example
- Added caveats section

---

## Proposed Pull Requests

| # | PR Name | Key Commits |
|---|---------|-------------|
| 1 | `ansible-lint: code quality fixes` | All lint fixes |
| 2 | `refactor: align naming with upstream` | Roles, groups, variables, services |
| 3 | `feat: HA cluster methods (kube-vip, keepalived, etcd)` | All HA role work |
| 4 | `feat: configuration validation role` | config-check role |
| 5 | `refactor: simplify group_vars and inventory` | Dictionary vars, cleanup |
| 6 | `fix: reset role full uninstall` | Reset improvements |
| 7 | `feat: download role enhancements` | Channels, commits, bin dir |
| 8 | `feat: multi-OS platform support` | RPi, ArchLinux, RHEL |
| 9 | `feat: additional packages, apparmor, firewall` | Extra features |
| 10 | `feat: kubeconfig saved to localhost` | Kubeconfig management |
| 11 | `fix: idempotency and control flow` | site.yml, wait, state changes |
| 12 | `feat: k3s service memory variables` | Memory vars |

---

## Rebase Strategy

### Why `git rebase k3s-io/main` won't work

The fork diverged in **November 2021**. Since then:
- **160 upstream commits** have landed on `k3s-io/main`
- **213 commits** exist in the fork
- Upstream made structural changes that overlap the fork's work: renamed roles, reorganized
  playbooks, added `airgap` and `k3s_upgrade` roles, moved to `playbooks/` (plural), changed
  `inventory-sample.yml` format

A straight `git rebase` would produce hundreds of conflicts and is not viable.

---

### Recommended Approach: New Branch Per PR

Create a fresh branch from `k3s-io/main` for each logical PR, manually applying only the unique
changes from the fork. This is clean, reviewable, and avoids conflict noise.

```bash
git fetch k3s-io
git checkout -b pr/01-description k3s-io/main
# apply changes, commit
git push jss pr/01-description
```

---

### What Upstream Already Has (skip or adapt)

| Fork Change | Upstream Status |
|---|---|
| `server`/`agent` host group names | Already in upstream |
| `ansible.builtin.*` FQCN | Already in upstream |
| Truthy `true`/`false` values | Already in upstream |
| `reboot.yml`, `reset.yml`, `site.yml` | Already in upstream (under `playbooks/`) |
| `.ansible-lint`, `Vagrantfile`, `collections/` | Already in upstream |
| IPv6 forwarding conditional | Already in upstream |
| `api_endpoint` variable name | Already in upstream |
| `k3s_server` / `k3s_agent` role names | Already in upstream |

---

### What the Fork Uniquely Contributes (the actual PRs)

Work through these in order — earlier ones are less likely to conflict with later ones:

#### PR 1 — `fix: idempotency and control flow`
**Low conflict risk.** Upstream's `site.yml`/roles are structurally similar.
- Wait for control plane before agents
- Prevent unnecessary state changes
- Only reload k3s service when files change
- Register variable naming to avoid conflicts

#### PR 2 — `feat: reset role`
**No conflict** — upstream has no `roles/reset/` at all; only a bare `playbooks/reset.yml`.
- Full `k3s-uninstall.sh`-equivalent reset role
- `remove_packages` option
- Killall task file
- VIP interface cleanup hooks

#### PR 3 — `feat: kubeconfig saved to localhost`
**Low conflict risk.** Upstream's `k3s_server` role handles kubeconfig but doesn't save to localhost.
- Save kubeconfig as `playbook/cluster.conf`
- Save both `localhost` and `api_endpoint` variants

#### PR 4 — `feat: k3s service memory variables`
**Low conflict risk.** Additive to `k3s_server` defaults.
- Memory-related service variables

#### PR 5 — `feat: config validation role`
**Low conflict risk** — upstream has no `roles/config_check`.
- Required variable validation
- HA-specific checks
- Empty `group_vars/all.yml` guard

#### PR 6 — `feat: HA cluster methods (kube-vip, keepalived, etcd)`
**Medium conflict risk** — upstream `site.yml` and `k3s_server` role would need HA hooks added.
- `roles/ha_kube_vip` (updated to v0.6.3 + load balancer)
- `roles/ha_keepalived`
- `roles/ha_etcd`
- Configuration checks for `ha_cluster_vip` / `ha_cluster_method`
- Externally-provided VIP option
- Hooks in `site.yml` and reset

#### PR 7 — `feat: additional packages, AppArmor, firewall`
**Medium conflict risk** — touches `roles/prereq` which upstream has modified significantly.
- AppArmor checks
- Firewall exceptions
- Additional manifests/packages for servers and agents
- Broken iptables check

#### PR 8 — `feat: download role enhancements`
**High conflict risk** — upstream no longer has a standalone `download` role. Needs investigation
to determine if this is still relevant or has been absorbed into `k3s_server`.

---

### Suggested Workflow

```bash
# One-time setup
git fetch k3s-io

# For each PR
git checkout -b pr/NN-description k3s-io/main

# Extract the fork's unique changes for that group
git diff k3s-io/main..HEAD -- <relevant files>

# Apply manually or cherry-pick individual commits without committing
git cherry-pick -n <sha>

# Resolve conflicts, adapt to upstream structure, then commit and push
git push jss pr/NN-description
```

---

### Suggested PR Order

| # | Branch | Risk |
|---|---|---|
| 1 | `pr/01-idempotency-control-flow` | Low |
| 2 | `pr/02-reset-role` | Low |
| 3 | `pr/03-kubeconfig-localhost` | Low |
| 4 | `pr/04-k3s-memory-variables` | Low |
| 5 | `pr/05-config-validation-role` | Low |
| 6 | `pr/06-ha-cluster-methods` | Medium |
| 7 | `pr/07-additional-packages-apparmor-firewall` | Medium |
| 8 | `pr/08-download-role-enhancements` | High — investigate first |
