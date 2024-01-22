# About
The purpose of this project is to automate the process of Kubernetes cluster configuration. For example, if you prefer not to use single-host options like miniKube and seek a k8s cluster without incurring high costs on cloud providers like AWS or GCP, you can opt for any affordable VPS host. On the other hand you could recreate your cluster again and again, just reset your VM's and run Ansible playbook again. So if you want cheap and low costed cluster for development or study purposes this could be helpfull for you.

### Quick start
If you are familiar with Ansible and its usage, you can simply configure the **[inventory](./inventory/hosts.ini)** and [run](#running) the playbook.

### Compatible OS
Tested with:
* Ubuntu 22.04
* Debian 12

### Compatible architectures
* x86_64
* ARM

## Table of Contents

* **[Requirements](#requirements)**
  * [Ansible](#ansible)
    * [Installation](#ansible-installation)
      * [Python](#python-installation)
      * [Virtual environment](#virtual-environment)
      * [Ansible package](#ansible-package)
    * [Configuration](#ansible-configuration)
    * [Break down roles](#break-down-roles)
      * [Init](#init-role)
      * [Docker](#docker-role)
      * [Kubernetes](#kubernetes-role)
      * [Master](#master-role)
      * [Worker](#worker-role)
* **[Configuration](#configuration)**
  * [Hosts](#hosts)
  * [User](#user)
  * [Version](#version)
  * [Pod network CIDR](#pod-network-cidr)
  * [Pod network overlay](#pod-network-overlay)
* **[Running](#running)**

## Requirements
Ansible required.

## Ansible
[Ansible](https://github.com/ansible/ansible) is an open-source automation tool, or platform, used for IT tasks such as configuration management, application deployment, intra-service orchestration and provisioning.

### Ansible installation
Ansible is written in Python, so before installing Ansible, you should ensure that Python is installed on your system.

#### Python installation
If Python is not installed on your host you could use [pyenv](https://github.com/pyenv/pyenv) both for Python installation and creating virtual environment to avoid conflicts with others Python packages. You could check their README installation part for instructions of how to install it for your OS but most easy and quick way is using automated installation script:

    curl https://pyenv.run | bash

After that follow the installer instructions to add necessary commands to your shell rc file (``.bashrc``, ``.zshrc``, etc.)

Before installation any version of Python you need to install all system dependencies for successfully Python compilation. Find necessary for your OS [here](https://github.com/pyenv/pyenv/wiki).

To get list of available Python versions run:

    pyenv install -l

Choose one of the latests stable. For example 3.11.5. To install:

    pyenv install 3.11.5

Now, you are able to create a virtual environment with the installed version of Python.

#### Virtual environment
If pyenv is installed then create virtual environment. For example: 

    pyenv virtualenv 3.11.5 ansible
    
where ```3.11.5``` is version of Python that you installed with pyenv and ```ansible``` is a name for virtual environment. 

Activate virtual environment with command: 

    pyenv activate ansible

Now, everything is ready for the Ansible installation.

#### Ansible package

When virtual environment activated run command to install ansible package with ``pip``:

    pip install ansible


At this moment Ansible is installed. But don't forget to activate your virtual anvironment every time when you want to use ansible if you closed your terminal before.

### Ansible configuration
If you need any specific configuration of Ansible you could edit ``ansible.cfg`` file in the root of repository.
By default, there are two variables preconfigured there:

| Name | Preconfigured value |
| --- | --- |
| forks | 10 |
| host_key_checking | False |

``forks`` - number of simultaneos Ansible sessions, how many hosts you can provision with Ansible in parallel. 

``host_key_checking`` - should or no Ansible check and warning host SSH key fingerprint. Leave it in False if you don't want handle ``known_hosts`` every time you rebuild your hosts.

### Break down roles
To better understanding what is going on under the hood of Ansible playbook let's break down Ansible roles that is used. In common, all roles just make the same steps that provided in Kubernetes documentation of how to configure hosts, init cluster and add worker nodes to it.

#### Init role
The very first and basic role that makes initial hosts preparation. Not only specific for Kubernetes but at common.

| Task | Description |
| --- | --- |
| **[hostname](./roles/init/tasks/hostname.yml)** | Change hostname of the machine to the hostname from Ansible inventory file |
| **[update](./roles/init/tasks/update.yml)** | Upgrade all system packages to latest versions. You could change upgrade type **[here](./roles/init/defaults/main.yml)** |
| **[packages](./roles/init/tasks/packages.yml)** | Install package manager related packages necessary to add aditional repositories |
| **[ssh](./roles/init/tasks/ssh.yml)** | Disable password authentication for SSH. If you need password authentication then comment this task **[here](./roles/init/tasks/main.yml)** |
 **[swap](./roles/init/tasks/swap.yml)** | Disable SWAP. Comment SWAP mount in ``/etc/fstab``. Remove SWAP file if exists |
| **[ntp](./roles/init/tasks/ntp.yml)** | Install NTP daemon. Ensures it's running. |

#### Docker role
This role is needed to add necessary repositories, install ``Docker`` and ``Containerd`` and configure containerd for Kubernetes.

| Task | Description |
| --- | --- |
| **[repositories](./roles/docker/tasks/repositories.yml)** | Add necessary Docker repository |
| **[docker](./roles/docker/tasks/docker.yml)** | Install Docker |
| **[containerd](./roles/docker/tasks/containerd.yml)** | Configure necessary Linux kernel modules. Install Containerd and configure it. Make sure it running |

#### Kubernetes role
This role to configure hosts with Kubernetes specific options, add necessary repositories and install Kubernetes packages - ``kubeadm``, ``kubelet``, ``kubectl``.

| Task | Description |
| --- | --- |
| **[sysctl](./roles/kubernetes/tasks/sysctl.yml)** | Configure network related sysctl parameters necessary for Kubernetes networking |
| **[repositories](./roles/kubernetes/tasks/repositories.yml)** | Add necessary Kubernetes repositories |
| **[packages](./roles/kubernetes/tasks/packages.yml)** | Install Kubernetes packages: kubeadm, kubelet, kubectl |
| **[services](./roles/kubernetes/tasks/services.yml)** | Restart kubelet service and make sure it is running |

#### Master role
This role to Initialize Kubernetes cluster and deploy pod network. For master node only.

| Task | Description |
| --- | --- |
| **[init](./roles/master/tasks/init.yml)** | Initialize Kubernetes cluster. Save worker join command as variable |
| **[network](./roles/master/tasks/network.yml)** | Deploy pod network |

#### Worker role
This role to run cluster join command on every worker node. Join command is taking from variable saved while master role execution.

| Task | Description |
| --- | --- |
| **[join](./roles/worker/tasks/join.yml)** | Join every worker node to cluster |

## Configuration
The mandatory only configuration is inventory file hostnames or IP addresses for nodes. All other options have default values.

#### Hosts
Put your hosts for master and worker nodes into **[inventory](./inventory/hosts.ini)** file. At this moment HA (High availability) is not implemented, so use 1 master node and any number of worker nodes.

#### User
For future implementation...

#### Version
For future implementation...

----

#### Pod network CIDR
For future implementation...

#### Pod network overlay
For future implementation...

## Running
When all hosts configured in inventory file you could run playbook:

    ansible-playbook playbook.yml -i inventory/hosts.ini

Once finished, your cluster will be ready to use. Maybe you should wait a little bit for all nodes ``Ready`` status. Usually it not takes a lot of time.

Don't forget:

    export KUBECONFIG=/etc/kubernetes/admin.conf

## License

MIT