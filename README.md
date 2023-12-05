# Build a Kubernetes cluster using *k3s* with *ansible*

Author: <https://github.com/itwars>\
Current Maintainer: <https://github.com/dereknola>

## Introduction to *k3s-ansible*

The goal of *k3s-ansible* is to easily install a Kubernetes cluster on a variety of
operating systems running on machines with different architectures.
In general, users of *k3s-ansible* should only need to create/edit one file: `inventory.yml`.

All you need to get started is a list of IP addresses for the hosts that you want to
participate in the cluster and a username that has password-less *ssh* access to all
those hosts.  That's it!
No need to futz with lots of settings and variables (unless you like that sort of thing;
then, have at it).

**NOTE: HA, any method, is broken at the moment.**

And, to setup an HA cluster, you need one more IP address - not of a host,
but for your cluster virtual IP address.
You don't need to know how to setup a clustering solution since *k3s-ansible* does it for you.
But, for HA, you just need at least three hosts.

The intention is for *k3s-ansible* to support what *k3s* supports.\
Here is what has been tested (:heavy_check_mark:) with *k3s-ansible*.

| Operating System | amd64 | arm64 | armhf |
| :--------------- | :---: | :---: | :---: |
| Debian           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Ubuntu           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| CentOS           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| RedHat           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| ArchLinux        | implemented | implemented | implemented |

## System requirements

- The deployment environment must have *ansible* v2.4.0+.
- Hosts in the cluster must have password-less *ssh* access.
- HA requires at least three hosts.

##  Caveats

- *k3s-ansible* will overwrite an existing *k3s* installation on the hosts.
- *k3s-ansible* will overwrite the `.kube` directory of the `ansible_user` specified on each server.
- An HA configuration using *keepalived* will overwrite an existing *keepalived* configuration.

## Usage

1. Create a new cluster definition based on the `inventory-examples/sample.yml` file.

```bash
cp inventory-examples/sample.yml inventory.yml
```

2. Edit `inventory.yml` to include the hosts that will make up your new cluster and
add variables to customize the install to best suit your environment.
See, `inventory-examples/README.md` for more details.\
For example:

```bash
---
k3s_cluster:
  children:
    server:
      hosts:
        192.168.1.26:
    agent:
      hosts:
        192.168.1.34:
        192.168.1.39:
        192.168.1.16:
        192.168.1.32:

  vars:
    ansible_user: debian
    cluster_config: cluster.conf
    install_k3s_channel: 'latest'
```

3. Provision your new cluster.

```bash
ansible-playbook playbook/site.yml
```

## Kubeconfig

To get access to your new **Kubernetes** cluster, just use the generated kube configuration file.

```bash
kubectl --kubeconfig playbook/cluster.conf ...
```

## Local Testing

A Vagrantfile is provided that provision a 5 nodes cluster using Vagrant (LibVirt or Virtualbox as provider). To use it:

```bash
vagrant up
```

By default, each node is given 2 cores and 2GB of RAM and runs Ubuntu 20.04. You can customize these settings by editing the `Vagrantfile`.

## High Availability
*k3s-ansible* can now configure a high-availability (HA) cluster.
If you enable HA (**ha_enabled**), the playbook will setup an embedded database using *etcd*.
HA requires at least version **v1.19.5+k3s1** and an odd number of servers (minimum of three).
See the [HA-embedded documentation](https://rancher.com/docs/k3s/latest/en/installation/ha-embedded/) for more details.

HA expects that there is a virtual IP (**ha_cluster_vip**) in front of the *control-plane* servers.
A few methods have been implemented to provide and manage this VIP.
See `inventory-examples/sample-ha.yml` for my example HA setup on my Turing Pi v1.
See `inventory-examples/README.md` for more details on variables.

