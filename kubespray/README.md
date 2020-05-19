# Versioning
Last Update: 2020-05-19 11:03

# Install a K8s cluster with Kubespray

## Initialization on EC2 instances
```
# Change hostnames of the machines
sudo hostnamectl set-hostname k8s-c2-node2

# Update and upgrade package repo
sudo apt update; sudo apt upgrade -y

# Install dependencies for Illumio VEN
sudo apt install -y curl net-tools dnsutils uuid-runtime ipset libnfnetlink0 libmnl0 libcap2 libgmp10 sed

# Install useful tools for network debugging
sudo apt install -y open-vm-tools git wireshark tcpdump nmap socat

# Upgrade Ubuntu Linux from 16.04 LTS > 18.04 LTS (required for Illumio VEN support on Ubuntu)
sudo do-release-upgrade
```

## Install prerequisites

### Ansible Server
```
# Install Ansible requirements on the Ansible Server
sudo apt-add-repository ppa:ansible/ansible

# Update packge repo
sudo apt-get update

# Installa Ansible, Python and PIP
sudo apt install -y ansible python-pip python3-pip python-netaddr python3-netaddr

# Install Jinja2
pip2 install jinja2 --upgrade

# Disable Firewall
sudo ufw disable
```

### Kubernetes nodes
```
# Enable ipv4 forwarding
sudo sysctl -w net.ipv4.ip_forward=1 <<< DOES NOT WORK ON UBUNTU IN AWS, NEED TO MODIFY THE SYSCTL CONFIG FILE!!!
```

## Change bash_profile on Ansible host

```
$ vim .bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# Start an SSH agent with the AWS keys
if [ -z "$SSH_AUTH_SOCK" ] ; then
    eval `ssh-agent`
    ssh-add ~/aws_demo_sales_new.pem
fi
```

## Install the requirements

### On each Node, change hostname, re-generate the machine-id and reboot the host (Bare-metal deployment)
```
sudo su
hostnamectl set-hostname k8s-cX-nodeY
systemd-machine-id-setup <<< DOES NOT WORK ON UBUNTU IN AWS!!!
reboot
```

### Remove current swap spaces created
`swapoff -a`

### Add this module at the end of /etc/modules
```
br_netfilter
```

### Uncomment net.ipv4.ip_forward statement in /etc/sysctl.conf
```
net.ipv4.ip_forward=1
```

### Add at the end of /etc/sysctl.conf
```
# Kubernetes requirements
net.bridge.bridge-nf-call-iptables=1
```

### Apply your changes (or reboot)
```
# Enable ipv4 forwarding
sysctl -p /etc/sysctl.conf
```

## Configure Kubespray

### Upgrade pip, clone Kubespray repository, and install the requirements
```
sudo pip3 install --upgrade pip
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/
sudo pip install -r requirements.txt
```

### Create a directory for the new cluster
```
cp -rpvf inventory/sample/ inventory/mycluster-1/
```

### Generate a new nodes inventory
```
declare -a IPS=(IP1 IP2 IP3 IP4 IP5 IP6)
CONFIG_FILE=inventory/mycluster-1/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

### Customize Kubernetes cluster config
```
# Customize nodes roles
vim inventory/mycluster-1/hosts.yml

# Customize etcd (optional)
vim inventory/mycluster-1/group_vars/etcd.yml

# Customize cluster informations (IP subnets, DNS, etc.)
vim inventory/mycluster-1/group_vars/k8s-cluster/k8s-cluster.yml

# Customize network plugins
vim inventory/mycluster-1/group_vars/k8s-cluster/k8s-net-*.yml

# Customize add-ons in the cluster
vim inventory/mycluster-1/group_vars/k8s-cluster/addons.yml

# Customize additional components (Proxy, external DNS, Load Balancer(s), container runtimes, providers (aws, azure, vsphere, etc.))
vim inventory/mycluster-1/group_vars/all/*.yml
```

## Deploy Kubernetes using Kubespray
```
ansible-playbook -i inventory/mycluster-1/hosts.yml  --become --become-user=root cluster.yml
```

### Error handling during deployment
Last time I deployed a cluster, I encountered an error related to the version of Docker. 
```
RUNNING HANDLER [container-engine/docker : Docker | wait for docker] *************************************************************************************************************************
fatal: [node4]: FAILED! => {"attempts": 20, "changed": true, "cmd": ["/usr/bin/docker", "images"], "delta": "0:00:00.036111", "end": "2020-05-19 17:36:09.310544", "msg": "non-zero return code", "rc": 1, "start": "2020-05-19 17:36:09.274433", "stderr": "Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39", "stderr_lines": ["Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39"], "stdout": "", "stdout_lines": []}
fatal: [node3]: FAILED! => {"attempts": 20, "changed": true, "cmd": ["/usr/bin/docker", "images"], "delta": "0:00:00.041190", "end": "2020-05-19 17:36:09.542005", "msg": "non-zero return code", "rc": 1, "start": "2020-05-19 17:36:09.500815", "stderr": "Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39", "stderr_lines": ["Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39"], "stdout": "", "stdout_lines": []}
fatal: [node2]: FAILED! => {"attempts": 20, "changed": true, "cmd": ["/usr/bin/docker", "images"], "delta": "0:00:00.045132", "end": "2020-05-19 17:36:09.586605", "msg": "non-zero return code", "rc": 1, "start": "2020-05-19 17:36:09.541473", "stderr": "Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39", "stderr_lines": ["Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39"], "stdout": "", "stdout_lines": []}
fatal: [node1]: FAILED! => {"attempts": 20, "changed": true, "cmd": ["/usr/bin/docker", "images"], "delta": "0:00:00.043148", "end": "2020-05-19 17:36:09.628260", "msg": "non-zero return code", "rc": 1, "start": "2020-05-19 17:36:09.585112", "stderr": "Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39", "stderr_lines": ["Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39"], "stdout": "", "stdout_lines": []}

NO MORE HOSTS LEFT ***************************************************************************************************************************************************************************

PLAY RECAP ***********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node1                      : ok=109  changed=16   unreachable=0    failed=1    skipped=165  rescued=0    ignored=0
node2                      : ok=91   changed=16   unreachable=0    failed=1    skipped=156  rescued=0    ignored=0
node3                      : ok=91   changed=16   unreachable=0    failed=1    skipped=156  rescued=0    ignored=0
node4                      : ok=90   changed=16   unreachable=0    failed=1    skipped=157  rescued=0    ignored=0
```

To fix such issue, we can force Kubespray to install a specific version of docker on the OS. Check the versions of docker available in file `roles/container-engine/docker/vars/ubuntu-amd64.yml`.

```
---
docker_kernel_min_version: '3.10'

# https://download.docker.com/linux/ubuntu/
docker_versioned_pkg:
  'latest': docker-ce
  '1.11': docker-engine=1.11.2-0~{{ ansible_distribution_release|lower }}
  '1.12': docker-engine=1.12.6-0~ubuntu-{{ ansible_distribution_release|lower }}
  '1.13': docker-engine=1.13.1-0~ubuntu-{{ ansible_distribution_release|lower }}
  '17.03': docker-ce=17.03.2~ce-0~ubuntu-{{ ansible_distribution_release|lower }}
  '17.09': docker-ce=17.09.0~ce-0~ubuntu-{{ ansible_distribution_release|lower }}
  '17.12': docker-ce=17.12.1~ce-0~ubuntu-{{ ansible_distribution_release|lower }}
  '18.06': docker-ce=18.06.2~ce~3-0~ubuntu
  '18.09': docker-ce=5:18.09.7~3-0~ubuntu-{{ ansible_distribution_release|lower }}
  '19.03': docker-ce=5:19.03.7~3-0~ubuntu-{{ ansible_distribution_release|lower }}
  'stable': docker-ce=5:18.09.7~3-0~ubuntu-{{ ansible_distribution_release|lower }}
  'edge': docker-ce=5:19.03.7~3-0~ubuntu-{{ ansible_distribution_release|lower }}
```

Update the Kubernetes config file to use the version chosen in file `inventory/mycluster-1/group_vars/k8s-cluster/k8s-cluster.yml`:
```
## Settings for docker runtime (only used when container_manager is set to docker)
docker_version: "19.03"
```

Save the file and re-run the ansible script.

## Install kubectl client
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

## Build your own private docker registry (Optional)
Follow the steps here: https://www.exoscale.com/syslog/setup-private-docker-registry/
