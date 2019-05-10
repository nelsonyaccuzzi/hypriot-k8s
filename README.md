# Hypriot Kubernetes Cluster

Create a Kubernetes Cluster using Raspberry Pi'si, HypriotOS, and KubeAdm.

## Prerequisites

- Raspberry Pi 3

With KubeAdm you only need one node (1 master) to start playing with K8s, so feel free to add up any number of nodes you want. This playbook was tested with 3 nodes (1 master and 2 workers)

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) 2.7 or higher

Ansible is a powerful configuration manager that is used here to bootstrap the K8s cluster.

- [HypriotOS](https://github.com/hypriot/image-builder-rpi/releases) v1.7 image or higher

HypriotOS is a Debian based distribution for Raspberry Pi. It preloads docker and optimize it to make it easy to use. It also bring support for cloud-init configs.

- [Hypriot Flash Command Line Script](https://github.com/hypriot/flash#installation)

Hypriot has his own flash tool that allow to preconfigure users, ssh keys, hostname, etc. using cloud-init config files.

> Ansible and Hypriot flash tool don't work on Windows Workstations

