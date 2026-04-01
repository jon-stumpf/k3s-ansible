# Source Plan — rebase-upstream

This file tracks the state of `rebase-upstream` on `jon-stumpf/k3s-ansible`.
It is the **source of truth** for what unique changes exist and what has been ported
to PR branches in `../k3s-ansible/`.

See `../k3s-ansible/PLAN.md` for PR submission status.

---

## Critical Notes

### playbook/ → playbooks/ rename (`4819dfb`)
Upstream (`k3s-io/main`) uses `playbooks/` (plural). This branch adopted that in `4819dfb`.
All PR branches in `k3s-ansible` must be based on `k3s-io/main` (which already has `playbooks/`),
so this rename does NOT need to be ported — just be aware the paths differ from older commits.

### New ansible-lint rules
Commits `92a1c5d`, `6b8cdfa`, `697565c`, `29e0a0c`, `2bf6db8`, `cc599a5` fix issues introduced
by newer `ansible-lint` versions. These fixes must be applied to PR branches before submission
(upstream CI will run `ansible-lint`).

---

## PR Groups — Source Commits

Porting status: `[x]` = ported to PR branch, `[ ]` = not yet ported, `[-]` = skip (upstream already has it)

---

### PR 1 — `fix: idempotency and control flow` — **PORTED / MERGED** (#514)

| Status | SHA | Message |
|---|---|---|
| [x] | `dc91550` | Eliminated unnecessary/redundant fact gathering |
| [x] | `635240f` | Prevented unnecessary tasks that caused state changes on hosts |
| [x] | `12b7b3f` | Fixed tasks that reported changes when there weren't |
| [x] | `6d94a87` | Made playbook/site.yml reentrant (for non-HA) |
| [x] | `46ba6e6` | Wait for control plane should only happen once |
| [x] | `c139d76` | Added wait for control-plane before configuring agents |
| [x] | `5ccaad4` | Increased wait time for control-plane to 60 seconds |
| [x] | `d3c42a8` | Updated register variables to avoid potential conflicts |
| [x] | `1f7b031` | Removed unnecessary task that always threw an error |

Ported as commits `4416386`, `598f322` on `pr/01-idempotency-control-flow`. Merged as PR #514.

---

### PR 2 — `refactor: group_vars reorganization using dictionaries` — **PARTIALLY PORTED**

PR branch `pr/02-group-vars-reorganization` in `k3s-ansible` has the initial 4 commits.
The following source commits have NOT yet been ported.

#### Already ported (initial dictionary structure)

| Status | SHA | Message |
|---|---|---|
| [x] | `c8ba198` | Cleaned up playbook/group_vars/all.yml by using dictionaries |
| [x] | `bfef60b`* | feat: introduce defaults.dir and k3s.dir dictionaries |
| [x] | `2c4bed2`* | feat: introduce defaults.cluster and k3s.cluster dictionaries |
| [x] | `546bfe2`* | feat: introduce k3s.version and k3s.args dictionaries |
| [x] | `ad59fcf`* | Fixed: dictionary entries cannot use default(omit) |

*SHA is in `k3s-ansible` PR branch, not directly in this repo.

#### Not yet ported (extended dictionary work — new commits)

| Status | SHA | Message |
|---|---|---|
| [ ] | `883df05` | Renamed defaults.cluster.first to defaults.cluster.endpoint |
| [ ] | `70e76c9` | Renamed k3s.cluster.address to k3s.cluster.endpoint |
| [ ] | `4067455` | Replaced ha_enabled with k3s.cluster.enabled in the codebase |
| [ ] | `e50bfe6` | Make sure only one of k3s.version.commit or k3s.version.tag are set |
| [ ] | `3f1dcbd` | Added k3s.flags to dictionary |
| [ ] | `48c0604` | Added k3s.cluster.subnet_mask |
| [ ] | `8d8242a` | Cleaned up roles/*/defaults/main.yml |
| [ ] | `d73102a` | Added the missing variables used in playbooks/group_vars/all.yml |
| [ ] | `65abda0` | Renamed kube_dir to kube_config |
| [ ] | `ddb5ec5` | Renamed userdir to kubedir |
| [ ] | `7be37a0` | Renamed ha_k3s_token to cluster_token |
| [ ] | `00295fd` | Replaced instances of '/etc/systemd/system' with k3s.dir.systemd |
| [ ] | `7646149` | Replace hardcoded instances of .kube directory |
| [ ] | `35c5e3c` | Replaced '/etc/rancher/k3s' with k3s.dir.etc |
| [ ] | `bc0cbeb` | Implemented the check for additional k3s services |

#### New ansible-lint fixes (must be included in PR 2 for CI to pass)

| Status | SHA | Message |
|---|---|---|
| [ ] | `92a1c5d` | ansible-lint: Fix var-naming[no-role-prefix] |
| [ ] | `6b8cdfa` | ansible-lint: Fix name[casing] |
| [ ] | `697565c` | ansible-lint: Fix FQCN |
| [ ] | `29e0a0c` | ansible-lint: Fix line-length |
| [ ] | `2bf6db8` | Replace octal modes with symbolic modes |
| [ ] | `cc599a5` | ansible.builtin.systemd → ansible.builtin.systemd_service |

---

### PR 3 — `feat: kubeconfig saved to localhost` — **NOT PORTED**

| Status | SHA | Message |
|---|---|---|
| [ ] | `6915b5f` | Added capability to copy the master kubeconfig to localhost in cluster.conf |
| [ ] | `aa0d4d3` | Moved update to cluster config to localhost |
| [ ] | `9402475` | Saved localhost and api_endpoint versions of the kubeconfig on each server |

---

### PR 4 — `feat: k3s service memory variables` — **NOT PORTED**

| Status | SHA | Message |
|---|---|---|
| [ ] | `174cd8a` | Added memory variables for the k3s service |

---

### PR 5 — `fix: reset role mirroring k3s-uninstall.sh` — **PORTED**

| Status | SHA | Message |
|---|---|---|
| [x] | `21a4ba9` | FIX #159: Updated roles/reset to match k3s-uninstall.sh |
| [x] | `99e8b38` | FIX #160: Moved k3s_server_location to group_vars/all.yml |
| [x] | `b3e2df8` | FIX #161: Updated pkill to use k3s_server_location |
| [x] | `0196954` | Addressed errors from k3s_server_location |
| [x] | `4357ad9` | FIX #162: Fixed errors with symlink'd commands |
| [x] | `a67e52d` | FIX #163: Removed ~/.kube/config on reset |
| [x] | `c2cb97e` | Removed ~/.kube directory, not just the config |
| [x] | `4a8adcf` | Ensured remove_packages is treated as boolean |
| [x] | `db52cc9` | Remove unnecessary node-token tasks |
| [x] | `38162f6` | Split out k3s-killall tasks into separate task file |
| [x] | `6790e39` | Broke up reset role in reset/download and reset/k3s |
| [x] | `2394e06` | Remove cluster VIP from interface during reset |
| [x] | `c946167` | Renamed keep_binaries to remove_packages |

Ported as commit `95581f7` on `pr/05-reset-role`. Not yet submitted.

---

### PR 6 — `feat: config validation role` — **NOT PORTED**

| Status | SHA | Message |
|---|---|---|
| [ ] | `6a9a02b` | Added roles/config-check |
| [ ] | `63318f2` | k3s_token must be defined |
| [ ] | `f42d11e` | Handled the case of an empty group_vars/all.yml file |
| [ ] | `1b5278f` | Added configuration checks for ha_cluster_vip and ha_cluster_method |

---

### PR 7 — `feat: HA cluster methods (kube-vip, keepalived, etcd)` — **NOT PORTED**

#### Core HA role commits

| Status | SHA | Message |
|---|---|---|
| [ ] | `3030e32` | Support HA mode with embedded DB |
| [ ] | `5db0830` | Added ha_enabled flag for HA embedded database using etcd |
| [ ] | `4a27b19` | Moved k3s-init tasks to a separate file |
| [ ] | `93184c9` | Converted k3s-init.yml to a role, ha/etcd |
| [ ] | `7b43a57` | Added cluster VIP method: externally provided cluster VIP |
| [ ] | `808bb14` | Added cluster VIP method: kube-vip |
| [ ] | `50d28e5` | Converted kube-vip cluster method to a role |
| [ ] | `5041688` | Turned kube-vip cluster method into roles |
| [ ] | `159841b` | Updated kube-vip to v0.6.3; Added kube-vip load-balancer |
| [ ] | `1577aa2` | Implemented keepalived cluster method as a role |
| [ ] | `1f6606a` | Cleaned up keepalived_* variables with a dictionary |
| [ ] | `928dc8d` | Fixed HA method keepalived |
| [ ] | `fd9dce4` | Fixed roles k3s/server and reset to use HA cluster method roles |
| [ ] | `b15fb8b` | Changed default HA cluster method to kube-vip |
| [ ] | `1ed9386` | Added cleanup to the install for kube-vip |

#### New HA bug fixes (include in this PR)

| Status | SHA | Message |
|---|---|---|
| [ ] | `b0b8ef4` | Fixed bug in k3s-init service name |
| [ ] | `b5acfdc` | Fixed duplicate name: in roles/ha_etcd/tasks/main.yml |
| [ ] | `f47c749` | Fixed bug in roles/ha_etcd/default/main.yml |
| [ ] | `ad90f91` | Fixed bug in processing 'ip -j link show' |
| [ ] | `3e1b070` | Added report_node_token |
| [ ] | `e1125ca` | Avoid some tasks when in check_mode |

---

### PR 8 — `feat: additional packages, AppArmor, firewall` — **NOT PORTED**

| Status | SHA | Message |
|---|---|---|
| [ ] | `768acb9` | Add AppArmor checks |
| [ ] | `8326557` | Allow firewall exceptions |
| [ ] | `a556128` | Check for broken iptables |
| [ ] | `a0cb432` | Add dependent Ubuntu packages |
| [ ] | `8211f30` | Added ability to install additional manifests and packages |
| [ ] | `75b53c8` | Add optional config file |

Note: `roles/prereq/` in upstream has changed significantly — diff carefully.

---

### PR 9 — `feat: download role enhancements` — **NOT PORTED**

| Status | SHA | Message |
|---|---|---|
| [ ] | `244e6a5` | Refactored roles/download and implemented commit vs version |
| [ ] | `97830ee` | Added support to change bin directory; Added ctr to symlinks |
| [ ] | `3b8842f` | Added capability to download version from a channel (e.g., stable) |
| [ ] | `7b76442` | Added k3s_commit variable |
| [ ] | `9630975` | Updated roles/download to use blocks for clarity |
| [ ] | `d43772e` | Moved roles/download/vars to roles/download/defaults |

**High risk** — upstream no longer has a standalone `download` role. Investigate before porting.

---

## Upstream Alignment Commits (rebase-upstream maintenance only, do NOT port)

These commits keep `rebase-upstream` aligned with `k3s-io/main` structural changes.
They do NOT need to be ported because PR branches are created from `k3s-io/main` directly.

| SHA | Message |
|---|---|
| `4819dfb` | Renamed playbook/ to playbooks/ to match upstream |
| `8150136` | Update roles/raspberrypi to upstream |

---

## Workflow

### Adding new commits to rebase-upstream

When `k3s-io/main` gets new features that conflict with or supersede fork changes:
1. Add commits to `rebase-upstream` here to reconcile
2. Update this PLAN.md — add commits to the appropriate PR group
3. Update porting status in the relevant PR group

### Porting a commit group to a PR branch

```bash
# In k3s-ansible/
git checkout pr/NN-description

# Cherry-pick from k3s-ansible-latest
git cherry-pick <sha>          # single commit
git cherry-pick <sha1>..<sha2> # range (exclusive..inclusive)

# Or apply files directly for manual porting
git checkout ../k3s-ansible-latest/rebase-upstream -- <file>
```

After porting, mark commits `[x]` in this file and update `../k3s-ansible/PLAN.md`.
