# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a fork of [k3s-io/k3s-ansible](https://github.com/k3s-io/k3s-ansible) — an Ansible playbook collection for building Kubernetes clusters using k3s.

| Name | URL | Branch |
|------|-----|--------|
| k3s-io (upstream) | https://github.com/k3s-io/k3s-ansible | `main` |
| jss (fork) | https://github.com/jon-stumpf/k3s-ansible | `rebase-upstream` |

The fork contains pull requests created years ago that addressed open issues and added new functionality. The original maintainer was unresponsive during that time. A new maintainer has since significantly updated the upstream codebase, overlapping some of that work, and closed the old PRs as out-of-date — but expressed interest in the changes. **The goal is to rebase the fork's changes onto the current upstream `main` and resubmit as new pull requests.**

`PLAN.md` tracks porting status per PR group. `COMMIT-ANALYSIS.md` groups commits into logical changes with proposed PR names.

> **Note:** HA method `kube-vip` is currently broken.

## Rebase Workflow

1. Analyze all commits in the `jss` fork relative to upstream `k3s-io/main`.
2. Group commits into logical, high-level changes with representative names (see `COMMIT-ANALYSIS.md`).
3. Propose new commits/PRs that cleanly apply those changes onto upstream `main`.
4. PR branches in `../k3s-ansible/` must be based on `k3s-io/main` (uses `playbooks/` plural).
5. Apply any required `ansible-lint` fixes before submitting upstream — upstream CI runs `ansible-lint`.

## Common Commands

```bash
# Install cluster
ansible-playbook playbooks/site.yml

# Uninstall cluster
ansible-playbook playbooks/reset.yml

# Rolling reboot
ansible-playbook playbooks/reboot.yml

# Dry run
ansible-playbook playbooks/site.yml --check

# Lint (mirrors CI)
yamllint .
ansible-lint

# Local testing with Vagrant (5 nodes: 3 server + 2 agent)
vagrant up
```

After install, the kubeconfig is at `playbooks/<cluster_config>` (default: `playbooks/config-cluster`):
```bash
kubectl --kubeconfig playbooks/config-cluster get nodes
```

## Architecture

### Variable System (`playbooks/group_vars/all.yml`)

Three top-level dictionaries drive all configuration:

- **`defaults`** — hardcoded fallback values; never reference these in roles directly
- **`k3s`** — the primary dict used throughout all roles; interpolates user inputs against `defaults`
- **`reference`** — static lookup tables (service names, valid HA methods, etc.)

User input variables (defined in `inventory.yml`) flow into `k3s.*` via Jinja2 expressions like:
```yaml
k3s.cluster.method: "{{ ha_cluster_method | default(defaults.cluster.method) }}"
```

All roles consume `k3s.*` variables — never use raw `defaults.*` or user inputs directly in role tasks.

### Inventory Structure

```
k3s_cluster:
  children:
    server:   # control-plane nodes (must be odd count for HA)
    agent:    # worker nodes
  vars:       # user input variables
```

User-configurable variables: `install_k3s_*` (map to k3s-install.sh env vars), `extra_*` (args/manifests/packages), `ha_*` (HA configuration), `cluster_token`, `kube_config`, `*_config`, `service_memory_*`, `report_*`, `remove_*`. Full reference in `INVENTORY.md`.

### Playbook Flow (`playbooks/site.yml`)

1. **All hosts**: `config_check` → `prereq` → `download` → `raspberrypi`
2. **Server group**: `k3s_server` (includes `ha_etcd` + `ha_keepalived`/`ha_kube_vip` when `ha_enabled: true`)
3. **Agent group**: `k3s_agent`

### Role Responsibilities

| Role | Purpose |
|------|---------|
| `config_check` | Pre-flight validation of required/conflicting variables |
| `prereq` | Kernel params, firewall (UFW/firewalld), packages |
| `download` | Fetch k3s binary from GitHub or custom URL |
| `raspberrypi` | RPi-specific: disable swap, cgroup limits |
| `k3s_server` | Install service, init HA, generate node token, write kubeconfig |
| `k3s_agent` | Install agent service, join cluster |
| `ha_etcd` | Initialize embedded etcd for HA |
| `ha_keepalived` | VRRP virtual IP via keepalived |
| `ha_kube_vip` | ARP virtual IP via kube-vip (currently broken) |
| `reset` | Stop services, remove binaries/config/packages |

### Key Directory Variables

Use `k3s.dir.*` (never hardcode paths):
- `k3s.dir.systemd` → `/etc/systemd/system`
- `k3s.dir.bin` → `/usr/local/bin`
- `k3s.dir.data` → `/var/lib/rancher/k3s`
- `k3s.dir.etc` → `/etc/rancher/k3s`

## CI

GitHub Actions (`.github/workflows/lint.yml`) runs `yamllint` + `ansible-lint` on PRs and pushes to `master`. YAML max line length is 120 (`.yamllint`).
