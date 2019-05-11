# Hypriot Kubernetes Cluster

Create a Kubernetes Cluster using Raspberry Pi's, HypriotOS, and KubeAdm.

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

## Steps

### Clone the repo

```
git clone https://github.com/nelsonyaccuzzi/hypriot-k8s.git
cd hypriot-k8s
```

### Flash SD Cards

Download the HypriotOS image from the [Official Repo Realease Page](https://github.com/hypriot/image-builder-rpi/releases).

```

wget https://github.com/hypriot/image-builder-rpi/releases/download/v1.10.1/hypriotos-rpi-v1.10.1.img.zip
unzip hypriotos-pi-v1.10.1.img.zip

```

Create a SSH Public/Private Key pair

```

ssh-keygen -f id_k8s

```
 
Create cloud-init config files for every node. The only thing that differs between the files is the hostname. 

Cloud-init look like

```
#cloud-config

hostname: rpi1
manage_etc_hosts: true

users:
  - name: pi
    primary-group: users
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users,docker
    ssh-import-id: None
    lock_passwd: true
    ssh-authorized-keys:
      - <id_k8s.pub_content>

package_upgrade: false

runcmd:
  - 'systemctl restart avahi-daemon'

```

Change the `ssh-authorized-keys` values with your own ssh public key and the hostname between files. Examples of cloud-init config files are in the flash dir

Flash the microSD cards

```

flash -u <cloud-init_file> <hypriotos_image>

```

Select the correct device where the microSD are mounted.

Repeat the command for every microSD card, changing the cloud-init file.

### Check Connectivity

> The same IP addresses and DNS names must be configured for this playbook to work. Configure your DHCP and DNS to deliver the same IP addresses and names to your nodes. 

Populate your inventory file

```
[master]
rpi1

[worker]
rpi2
rpi3
```

Check connectiviy

```

ansible all -m ping -i inventory --private-key id_k8s -u pi

```

### Create the K8s Cluster

Execute the playbook

```

ansible-playbook -i inventory main.yml

```

### Checking

Checking on nodes

```
kubectl get nodes

NAME   STATUS   ROLES    AGE   VERSION
rpi1   Ready    master   61m   v1.14.1
rpi2   Ready    <none>   60m   v1.14.1
rpi3   Ready    <none>   60m   v1.14.1

```

## TBD

- Multi Master configuration
- Static IP Address setting on cloud-init
- Hosts file setup on nodes
- Flash hostname parameter usage with cloud-init config files
- MetalLB and Ingress Controller deployment on setup
- Cluster Init and Setup playbook separation

## References

This playbook was created following this helpful repos

- [rak8s](https://github.com/rak8s/rak8s)
- [kubernetes-arm](https://github.com/carlosedp/kubernetes-arm)

