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
    1. [/worker](./create/worker/) - contains configurations to initialize and configure worker nodes

- What's in the [/control](./create/control/) directory?
    1. [cloud-init-template.j2](./create/control/cloud-init-template.j2): multipass supports cloud-init, this template is used to create the cloud-init config file
    1. [cloud-init.yaml](./create/control/cloud-init-template.j2): cloud init file for the control VMs
    1. [init-cluster-tasks.yaml](./create/control/init-cluster-tasks.yaml): tasks to initialize the cluster, configure the control nodes
    1. [kubeadm-config-template.j2](./create/control/kubeadm-config-template.j2: template for kubeadm config file
    1. [playbook.yaml](./create/control/playbook.yaml): playbook that calls `init-cluster-tasks.yaml`

- What's in the [/loadbalancer](./create/loadbalancer/) directory?
    1. [cloud-init-template.j2](./create/loadbalancer/cloud-init-template.j2): multipass supports cloud-init, this template is used to create the cloud-init config file
    1. [cloud-init.yaml](./create/loadbalancer/cloud-init.yaml): cloud init file for the load balancer VM
    1. [haproxy-cfg-template.j2](./create/loadbalancer/haproxy-cfg-template.j2): template for haproxy config file
    1. [playbook.yaml](./create/loadbalancer/playbook.yaml): playbook that calls `update-lb-tasks.yaml`
    1. [update-lb-tasks.yaml](./create/loadbalancer/update-lb-tasks.yaml): tasks to update the haproxy load balancer config

- What's in the [/main](./create/main/) directory?
    1. [create-vm-tasks.yaml](./create/main/create-vm-tasks.yaml): defines tasks to create the nodes according to config file and creates an inventory file to be consumed in a different ansible play
    1. [playbook.yaml](./create/main/playbook.yaml): defines the main ansible playbook - parses the config file, executes tasks in `create-vm-tasks.yaml` and run plays from other playbooks to configure an HA k8s cluster

- What's in the [/worker](./create/worker/) directory?
    1. [cloud-init-template.j2](./create/worker/cloud-init-template.j2): multipass supports cloud-init, this template is used to create the worker VMs cloud-init config file
    1. [cloud-init.yaml](./create/worker/cloud-init.yaml): cloud init file for the worker VMs    
    1. [join-workers-tasks.yaml](./create/worker/join-workers-tasks.yaml): tasks to join worker nodes to the cluster, configure the worker nodes
    1. [playbook.yaml](./create/worker/playbook.yaml): playbook that calls `join-workers-tasks.yaml`

- What's in the [/inventories](./inventories/) directory?
    1. [control.yaml](./inventories/control.yaml): inventory file for control nodes
    1. [loadbalancer.yaml](./inventories/loadbalancer.yaml): inventory file for load balancer nodes
    1. [worker.yaml](./inventories/worker.yaml): inventory file for worker nodes

- What's in the [/upgrade](./upgrade/) directory?
    1. [/control](./upgrade/control/) - contains configurations to upgrade control plane nodes
    1. [/worker](./upgrade/worker/) - contains configurations to upgrade worker nodes
    1. [playbook.yaml](./upgrade/playbook.yaml): playbook that calls playbooks from `/control` and `worker`

- What's in the [/control](./upgrade/control/) directory?
    1. [upgrade-control-tasks.yaml](./upgrade/control/upgrade-control-tasks.yaml): tasks to upgrade the control nodes
    1. [playbook.yaml](./upgrade/control/playbook.yaml): playbook that calls `upgrade-control-tasks.yaml`

- What's in the [/worker](./upgrade/worker/) directory?
    1. [upgrade-worker-tasks.yaml](./upgrade/worker/upgrade-worker-tasks.yaml): tasks to upgrade the worker nodes
    1. [playbook.yaml](./upgrade/worker/playbook.yaml): playbook that calls `upgrade-worker-tasks.yaml`

## Prerequisites
- Install [multipass](https://multipass.run/)
- Install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Create a kubernetes cluster

- How to create a Highly Available kubernetes cluster
    1. cd `/create/main`
    1. run `ansible-playbook --inventory ../../inventories/ playbook.yaml --extra-vars "configFile=1l3c3w.yaml"`

- How to update a Highly Available kubernetes cluster
    1. cd `/update`
    1. run `ansible-playbook --inventory ../inventories/ playbook.yaml`

- How to delete the cluster
    1. run `multipass delete -all` to delete all the VMs. ❗ it will delete all VMs, including the ones not created by ansible ❗
    1. run `multipass purge` to purge the VMs