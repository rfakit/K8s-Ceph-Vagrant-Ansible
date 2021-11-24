
# K8s-Ceph-Vagrant-Ansible

Deploying k8s multi master cluster alogside ceph as storageclass with the power of Vagrant and Ansible

## Tech Stack

**Tools:**  Ansible, Kubespray , Ceph-ansible , Vagrant , VirtualBox

**Final Deploy:**  K8s multi-master Cluster, Ceph rbd Storageclass , Mariadb Galera Cluster , Prometheus , Grafana

**IAC:**  Ansible , Vagrant

## Introduction

In this section we focus on the project which is described bellow

- Create multi-master kubernetes cluster using Vagrant in three VirtualBox VM (or multiple VMs)
- Install and configure CEPH as storage-class
- Deploy Nginx Ingress
- Install Prometheus, Grafana to k8s cluster
- Install three node mariadb galera cluster to k8s cluster
- Create monitoring dashboard for mariadb cluster
- Last but not Least, All above tasks must be done via any automation tools (In this project I use Ansible 💪)

## Prerequisite

**Minimum HardWare:**  A Linux System with Cpu 8 Core / 24G memory RAM (the more , the better)

**Software:**  Ansible , Vagrant , Virtualbox , Ansible community.general Collection

**OS:**  Any Redhat / Debian Based OS (In this project we use ubuntu20.04-3 lts)


## Prerequisites Installation

First make sure that all needed softwares are installed.

Insatll Ansible Vagrant and Virtualbox bassed on your OS that you're using. (it's recommended to use the latest version of these softwares that exsist in your OS official repo)

for install ansible collection, use the bellow command.

the only module that we are gonna use within `community.general` is `mail`, We use it to send email notification when Creating Cluster (k8s and ceph) encounter any errors.

```bash
  ansible-galaxy collection install community.general
```



## Documentation

This project is an Ansible role to meet those needs that describe at `Introduction` part.

### Main Line
This ansible role first create a set of vms in virtualbox via vagrant and provision Ceph cluster , then
create another set of vms and provision k8s cluster ,lastly we install GaleraCluster with helm package manager and monitor the status of Database via Prometheus and Grafana.

let's move on to some specific Explanation.

Within this section We describe the way that how i managed to implement these stacks.

lets start with `tasks` directory.

the `main.yaml` file Arrange the order of execution of tasks.

#### prep.yml
This playbook will install python3-pip which is used to install python3 modules for ceph-ansible and kubespray project.
then we clone the  ceph-ansible and kubespray git project to `/opt`

#### ceph.yml
This playbook will copy `vagrant_variables.yml.j2` to proper location (you will findout how to customize the project in `Customization` section )
then it start create and provision ceph cluster via `vagrant up` command, finally it update the inventory file to create ceph pool in ceph cluster,

#### ceph-create-pool.yml
This playbook will Create pool name `k8s` for kubernetes and save auth.key of client and admin user into variable to use it at next playbook.

#### k8s-cluster.yml
This playbook will update addons.yaml file with ceph information specificely auth.key . this file will be fed to kubespray to deploy ceph as storageclass.then it will copy other customization k8s parameter to proper location and run `vagrant up` command to create and provision the k8s cluster.

#### k8s-prom-mariadb.yml
This playbook will first update file kube-apiserver.yaml for fixing this issue  [unexpected error getting claim reference: selfLink was empty, can't make reference](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/issues/25).
then send value-*.yaml to k8s first node to deploy prometheus and mariadb into k8s cluster, these value-* files will be used as  customization for helm package manager.
finally we run helm install to provision prometheus and GaleraCluster.

#### k8s-ingress.yml
This playbook will install  ingress-nginx, this is the bare metal version because there isnt any cloud manager infrastructore to serve loadbalance Ip address so we have to use nodeport service for ingress.

#### k8s-grafana.yml
This playbook will copy datasource and dashboard files to server and create proper Configmaps and Secrets from these files to serve to grafana.
besides it cppys valu-grafana.yaml to customize the helm installtion of Grafana.

#### email.yml
This playbook will tasks to send email notification whenever the `vagrant up` command encounter any errors
## Customization

Before We start installation we can alter the parameters to setup the whole stacks bassed on our needs.

there are 3 directories that you can use to Customize the installation.
- defaults
- templates
- files

the most important directory is defaults. within the `defaults` directory we put some important.
most of the parameters are doing well but make sure you set `inventory_location` var to proper location.

`defaults` directory contains a file named `main.yaml` which consists variable for Ceph, K8S and Email.

Some Customization can be done inside the other 2 directories.

`template` contains of some files that the most important parameters of them can be eddited via `defaults` directory.

`files` contains Customization files for prometheus grafana and mariadb, with the directory of `group_vars` which is used by kubespray, I wanna remind that the most important parameters can be altered via `defaults/main.yaml` file.

## Installation

To start Installation first we clone the repo to proper location and follow the instruction,
```bash
  cd /opt
  git clone
```
For start deploying the stacks we use the bellow command.
 ```bash
  cd /opt/asd/
  ansible-playbook -i test/inventory test/test.yaml
```
Wait Until the Installation will be finished.

## Access Metrics

after Installation is completed It will show you the Nodeport that was asigned to nginx-ingress.
To access Grafana interface just use any Adress of nodes with the Nodeport shown at last task.
because we didnt handle the DNS setup due to implementation in private network, update the /etc/hosts file in your system to 
sth like this

 ```bash
   192.168.120.101 grafana.local
```

then access the web via the link http://grafana.local:{NODEPORT}
It will redirect you to /login page and the default grafana login access is admin:admin . 
after the first login you are forced to change the admin password.

I like to mention that 2 dashboards were added to grafana and prometheus datasource was configured . so you just need to open dashboards within the grafana and start digging the metrics.


## Contributing

Contributions are always welcome!

Feel free to fork it.
## License

[![MIT License](https://img.shields.io/apm/l/atomic-design-ui.svg?)](https://github.com/tterb/atomic-design-ui/blob/master/LICENSEs)

