# Kubernetes-on-Multipass
Kubernetes on multipass ubuntu VMs using ansible

## High level overview
- Use ansible to create a number of [multipass](https://multipass.run/) VMs
- Check out multipass [refence documentation](https://multipass.run/docs/reference) for more details on commands used
- Use kubeadm to orchestrate a kubernetes cluster on the VMs as control and worker nodes
- Use kubectl to interact with the cluster

## How it works
- What's in the [/create](./create/) directory?
    1. [/control](./create/control/) - contains configurations to initialize and configure control plane nodes
    1. [/loadbalancer](./create/loadbalancer/) - contains configurations to configure load balancer node
    1. [/main](./create/main/) - contains configurations to create all the nodes needed for a kubernetes cluster, including control, worker, and load balancer nodes

- What's in the [/control](./create/control/) directory?
    1. [init-first-ctrl-node-tasks.yaml](./create/control/init-first-ctrl-node-tasks.yaml): tasks to initialize the cluster, configure the control nodes
    1. [kubeadm-config-template.j2](./create/control/kubeadm-config-template.j2): template for kubeadm config file

- What's in the [/loadbalancer](./create/loadbalancer/) directory?
    1. [cloud-init-template.j2](./create/loadbalancer/cloud-init-template.j2): multipass supports cloud-init, this template is used to create the cloud-init config file
    1. [haproxy-cfg-template.j2](./create/loadbalancer/haproxy-cfg-template.j2): template for haproxy config file
    1. [configure-lb-tasks.yaml](./create/loadbalancer/configure-lb-tasks.yaml): tasks to configure the haproxy load balancer config

- What's in the [/main](./create/main/) directory?
    1. [cloud-init-template.j2](./create/main/cloud-init-template.j2): multipass supports cloud-init, this template is used to create the worker VMs cloud-init config file
    1. [create-vm-tasks.yaml](./create/main/create-vm-tasks.yaml): defines tasks to create the nodes according to config file and creates an inventory file to be consumed in a different ansible play
    1. [join-cluster-tasks.yaml](./create/main/join-cluster-tasks.yaml): tasks to configure & join nodes to the cluster
    1. [playbook.yaml](./create/main/playbook.yaml): defines the main ansible playbook - parses the config file, executes tasks in `create-vm-tasks.yaml` and run plays from other playbooks to configure an HA k8s cluster

- What's in the [/inventories](./inventories/) directory?
    1. [control.yaml](./inventories/control.yaml): inventory file for control nodes
    1. [loadbalancer.yaml](./inventories/loadbalancer.yaml): inventory file for load balancer nodes
    1. [worker.yaml](./inventories/worker.yaml): inventory file for worker nodes

- What's in the [/upgrade](./upgrade/) directory?
    1. [upgrade-node-tasks.yaml](./upgrade/upgrade-node-tasks.yaml): tasks to upgrade the control nodes
    1. [playbook.yaml](./upgrade/playbook.yaml): playbook that calls playbooks from `/control` and `worker`

## Prerequisites
- Install [multipass](https://multipass.run/)
- Install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Create a kubernetes cluster

- How to create a kubernetes cluster
    1. run `ansible-playbook create/playbook.yaml`

- How to upgrade a kubernetes cluster
    1. run `ansible-playbook upgrade/playbook.yaml`

- How to delete the cluster
    1. run `multipass stop --all` to stop all the multipass VMs
    1. run `multipass delete -all` to delete all the multipass VMs. ❗ FYI: will DELETE all multipass VMs ❗
    1. run `multipass purge` to purge the VMs