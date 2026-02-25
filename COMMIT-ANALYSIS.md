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
