# Hypriot Kubernetes Cluster

Create a Kubernetes Cluster using Raspberry Pi's, HypriotOS, and KubeAdm.

## Prerequisites

- Raspberry Pi 3/4

With KubeAdm you only need one node (1 master) to start playing with K8s, so feel free to add up any number of nodes you want. 


- Linux

Ansible only works under Linux. So if you are using a Windows workstation you can create a Linux Virtual Machine or use [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) 2.8 or higher

Ansible is a powerful configuration manager that is used here to bootstrap the K8s cluster.

- [HypriotOS](https://github.com/hypriot/image-builder-rpi/releases) v1.7 image or higher

HypriotOS is a Debian based distribution for Raspberry Pi. It preloads docker and optimize it to make it easy to use. It also bring support for cloud-init configs.

- Openshift Python Module

Ansible use it to operate with the Kubernetes Cluster. You can install it easily with `pip`

```

pip install openshift

```

## Steps

### Clone the repository

```
git clone https://github.com/nelsonyaccuzzi/hypriot-k8s.git
cd hypriot-k8s
```

### Complete the inventory

Edit the `inventory` and `vars` file with your variables and hosts

```
[master]
rpi-mst-01 ansible_host=192.168.0.211
rpi-mst-02 ansible_host=192.168.0.212
rpi-mst-03 ansible_host=192.168.0.213

[worker]
rpi-wrk-01 ansible_host=192.168.0.214
rpi-wrk-02 ansible_host=192.168.0.215
rpi-wrk-03 ansible_host=192.168.0.216

```

```

master_ip_address: 192.168.0.210
network_mask: 255.255.0.0
gateway_ip_address: 192.168.0.1
dns_ip_address: 192.168.0.35

cluster_address: k8s
kubernetes_version: 1.18.4

```

### Create cloud-init configs

```

ansible-playbook -i inventory -e "@vars" playbooks/configs.yml

```
After this, a folder name `user-data` will appear in this repo with the cloud-init files for each raspberry

### Download Image

Download the HypriotOS image from the [Official Repo Realease Page](https://github.com/hypriot/image-builder-rpi/releases).

```

wget https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.2/hypriotos-rpi-v1.12.2.img.zip
unzip hypriotos-pi-v1.12.2.img.zip

```

### Flash MicroSD Cards

You can use any tool you like to do this, but in the end you have to copy the content of each cloud-init file in the `user-data` file inside the /boot partition of each microSD card

If you are using Linux you can use the [Hypriot Flash Tool](https://github.com/hypriot/flash) to flash and copy the cloud-init in the same command, like this

```

flash -u <cloud-init_file> <hypriotos_image>

```

Select the correct device where the microSD are mounted.

Repeat the command for every microSD card, changing the cloud-init file.

### Check Connectivity

> The cloud-init configs configures the eth0 interface with the inventory specified ip address and network configuration, wlan0 support will be added later.

```

ansible all -m ping -i inventory --private-key keys/id_k8s -u pi

```
> Passwordless connections are also configured for the pi user and the keys are located in the `keys` folder

### Create the K8s Cluster

Execute the playbook

```

ansible-playbook -i inventory -e "@vars" main.yml

```

### Checking

Checking on nodes

```
kubectl get nodes

NAME         STATUS   ROLES    AGE   VERSION
rpi-mst-01   Ready    master   23h   v1.18.5
rpi-mst-02   Ready    master   23h   v1.18.5
rpi-mst-03   Ready    master   23h   v1.18.5
rpi-wrk-01   Ready    worker   23h   v1.18.5
rpi-wrk-02   Ready    worker   23h   v1.18.5
rpi-wrk-03   Ready    worker   23h   v1.18.5

```

### Cleanup Cluster

If you want to keep playing you can start over with the following command

```

ansible-playbook -i inventory playbooks/cleanup.yml

```

## References

This playbook was created following this helpful repos

- [rak8s](https://github.com/rak8s/rak8s)
- [kubernetes-arm](https://github.com/carlosedp/kubernetes-arm)

