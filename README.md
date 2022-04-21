# Kubernetes-on-Multipass
Kubernetes on multipass ubuntu VMs using ansible

## High level overview
- Use ansible to create a number of [multipass](https://multipass.run/) VMs
- Check out multipass [refence documentation](https://multipass.run/docs/reference) for more details on commands used
- Use kubeadm to orchestrate a kubernetes cluster on the VMs as control and worker nodes
- Use kubectl to interact with the cluster

## How it works
- What's in the /ansible directory?
    1. `1m3w.yaml`: defines cluster nodes configFile in yaml - 1 control, 3 workers
    1. `create-nodes-tasks.yaml`: defines tasks to create the nodes and creates an inventory file to be cosumed in a different ansible play
    1. `playbook.yaml`: defines the main ansible playbook - parses `1m3w.yaml`, executes tasks in `create-nodes-tasks.yaml` and run plays from other playbooks
    1. `/inventories`: contains the inventory file for the nodes, 1 file for control nodes, 1 file for worker nodes

- What's in the /control directory?
    1. `cloud-init-template.j2`: multipass supports cloud-init, this template is used to create the cloud-init config file
    1. `cloud-init.yaml`: cloud init file for the control VMs
    1. `init-cluster-tasks.yaml`: tasks to initialize the cluster, configure the control nodes
    1. `kubeadm-config-template.j2`: template for kubeadm config file
    1. `kubeadm-config.yaml`: renders kubeadm config file for the first control node
    1. `playbook.yaml`: playbook that calls `init-cluster-tasks.yaml`

- What's in the /worker directory?
    1. `cloud-init-template.j2`: multipass supports cloud-init, this template is used to create the worker VMs cloud-init config file
    1. `cloud-init.yaml`: cloud init file for the worker VMs    1. `join-workers-tasks.yaml`: tasks to join worker nodes to the cluster, configure the worker nodes
    1. `playbook.yaml`: playbook that calls `join-workers-tasks.yaml`


## Prerequisites
- Install [multipass](https://multipass.run/)
- Install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Run on your machine to create the cluster

- How to create the cluster
    1. cd `/ansible`
    1. run `ansible-playbook --inventory inventories/ playbook.yaml --extra-vars "configFile=<1m3w.yaml|3m3w.yaml>"`

- How to delete the cluster
    1. run `multipass delete -all` to delete all the VMs. ❗ it will delete all VMs, including the ones not created by ansible ❗
    1. run `multipass purge` to purge the VMs