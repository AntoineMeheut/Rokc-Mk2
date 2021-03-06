<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![GNU License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/AntoineMeheut/Rokc-Mk2">
    <img src="images/MacBook_Running_Virtual_Machine.svg" alt="LattePanda" width="200" height="150">
    <img src="images/openshift.png" alt="Openshift" width="150" height="150">
    <img src="images/kubernetes.png" alt="Kubernetes" width="150" height="150">
  </a>

  <h3 align="center">Deploying OpenShift Origin 3.11 on VM cluster</h3>

  <p align="center">
    Rokc Mark 2 was born out of a wall I ran into while trying to compile Openshift for ARM on Rokc-Mk1 project. It doesn't matter, his little brother is a new adventure of reflection, discovery, reading, training and sharing. By keeping the same technical objective to run a private cloud with Openshift, deploy applications and work on security to improve and share my knowledge on the Sec of DevSecOps. **With the experience gained on the Rokc-Mk1 project, I first decided to experiment with the project on VMs in order to be sure that it works before buying the server cards necessary for the installation on metal bar.**
    <br />
    <br />
    <a href="https://github.com/AntoineMeheut/Rock-Mk2/issues">Report Bug</a>
    ·
    <a href="https://github.com/AntoineMeheut/Rock-Mk2/issues">Request Feature</a>
  </p>
</p>


<!-- TABLE OF CONTENTS -->
* [Deploying OpenShift Origin on VM cluster](#deploying-openShift-origin-on-vm-cluster)
	* [Infrastructure Setup](#infrastructure-setup)
		* [Install Centos-7 on all your nodes](#install-centos-7-on-all-your-nodes)
		* [Installing cockpit tool on CentOS 7 for all nodes](#installing-cockpit-tool-on-centos-7-for-all-nodes)
			* [Install Cockpit](#install-cockpit)
			* [Install additional Cockpit packages](#install-additional-cockpit-packages)
			* [Enable Cockpit](#enable-cockpit)
			* [Add cockpit to firewall](#add-cockpit-to-firewall)
		* [Connect to your nodes interfaces](#connect-to-your-nodes-interfaces)
	* [Preparing Nodes](#preparing-nodes)
		* [Set the hostname for each corresponding node](#set-the-hostname-for-each-corresponding-node)
		* [Configure static ip](#configure-static-ip)
		* [Configure names resolution for all node](#configure-names-resolution-for-all-node)
	* [Openshift Origin installation](#openshift-origin-installation)
		* [Installation of OKD and Ansible](#installation-of-okd-and-ansible)
		* [Add docker to firewall](#add-docker-to-firewall)
		* [Now you can reboot your node](#now-you-can-reboot-your-node)
		* [Connect to your nodes cockpit interfaces](#connect-to-your-nodes-cockpit-interfaces)
		* [Deploying and starting Openshift Origin from master node](#deploying-and-starting-openshift-origin-from-master-node)
			* [Preparation on master only](#preparation-on-master-only)
				* [Creating an RSA key](#creating-an-rsa-key)
				* [Declare the target nodes for the key](#declare-the-target-nodes-for-the-key)
				* [Send the public-key to all the nodes](#send-the-public-key-to-all-the-nodes)
			* [Preparing the hosts file for Ansible](#preparing-the-hosts-file-for-ansible)
			* [Run prerequisites playbook](#run-prerequisites-playbook)
			* [Run deploy cluster playbook](#run-deploy-cluster-playbook)
		* [Useful commands to verify that it works](#useful-commands-to-verify-that-it-works)
			* [See the state of your nodes](#see-the-state-of-your-nodes)
			* [View status with labels](#view-status-with-labels)
			* [See the state of your pods](#see-the-state-of-your-pods)
			* [Create User Accounts for OKD console](#create-user-accounts-for-okd-console)
				* [Create a user account](#create-a-user-account)
				* [Restart OpenShift before going forward](#restart-openShift-before-going-forward)
		* [Access the OKD console](#access-the-okd-console)
* [Roadmap](#roadmap)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)
* [Acknowledgements](#acknowledgements)

<!-- ABOUT THE PROJECT -->
# Deploying OpenShift Origin on VM cluster
## Infrastructure Setup
| Hostname | IP Address | VM Disk | VM Memory | OS | Role |
| ---- | ---- | ---- | ---- | ---- | ---- |
| master.openshift.hal9000.com | 192.168.148.130 | 40 Gb | 8Gb | Centos-7 | Master Node |
| node1.openshift.hal9000.com | 192.168.148.131 | 40 Gb | 8Gb | Centos-7 | Worker Node 1 |
| node2.openshift.hal9000.com | 192.168.148.132 | 40 Gb | 8Gb | Centos-7 | Worker Node 2 |

To do this installation on virtual machines, on a laptop, you need a minimum of 32 Gb of memory.
We will use 3 x 8 Gb per virtual machine, for each of the 3 nodes and it will be necessary
to leave some free memory for your OS and the virtualization software that you will use.

I built this tutorial on a MacBook Pro using VMware Fusion, I think it is possible to achieve this
using VirtualBox, but I have not tested.

### Install Centos-7 on all your nodes
[CentOS-7-x86_64-Minimal-2003.iso](http://isoredirect.centos.org/centos/7/isos/x86_64/)

If you need detailed information on this phase, refer to the installation on target bar metal.

### Installing cockpit tool on CentOS 7 for all nodes
#### Install Cockpit
```
yum install -y cockpit
```
#### Install additional Cockpit packages
```
yum install -y cockpit-networkmanager cockpit-dashboard cockpit-storaged cockpit-packagekit
```
#### Enable Cockpit
```
systemctl enable --now cockpit.socket
```
#### Add cockpit to firewall
```
firewall-cmd --permanent --add-service=cockpit
firewall-cmd --reload
```
### Connect to your nodes interfaces
You should connect on root on all ur nodes.

* master [https://192.168.148.130:9090](https://192.168.148.130:9090)
* node1 [https://192.168.148.131:9090](https://192.168.148.131:9090)
* node2 [https://192.168.148.132:9090](https://192.168.148.132:9090)

## Preparing Nodes
### Set the hostname for each corresponding node
Master

```
hostnamectl set-hostname master.hal9000.com
```
Node1

```
hostnamectl set-hostname node1.hal9000.com
```
Node2

```
hostnamectl set-hostname node2.hal9000.com
```

### Configure static ip
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

Change BOOTPROTO from dhcp to static and add the following lignes to the file. Do that for each nodes with IPADDR 130 for master, 131 for node1 and 132 for node2.

Realy take care of the GATEWAY cause it depend on your virtualization software, for my VMware Fusion it's 192.168.148.2

Take care to keep the UUID of the file you are modifying, you can delete all lines except this one.

```
TYPE=Ethernet
PROXY_METHODE="none"
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME=ens33
DEVICE=ens33
ONBOOT=yes
PROXY_METHOD=none
IPADDR=192.168.148.130
PREFIX=24
NETMASK=255.255.255.0
GATEWAY=192.168.148.2
DNS1=8.8.8.8
UUID=c96bc909-188e-ec64-3a96-6a90982b08ad <keep your UUID already present in your file>
```

### Configure names resolution for all node
Configure /etc/hosts file for name resolution as following on all nodes.

```
vi /etc/hosts
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.148.130   master.openshift.hal9000.com  master
192.168.148.131   node1.openshift.hal9000.com  node1
192.168.148.132   node2.openshift.hal9000.com  node2
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

## Openshift Origin installation
### Installation of OKD and Ansible
On all Nodes, install OpenShift Origin 3.11 repository, Ansible and Docker.

For Ansible, version 2.6, 2.7, 2.8, 2.9 are provided from CentOS Repository, but Openshift-Ansible is not supported on 2.8 or later, so install Ansible 2.7

```
yum -y install centos-release-openshift-origin311 centos-release-ansible-27
yum -y install ansible openshift-ansible docker git pyOpenSSL
systemctl enable --now docker
```

### Add docker to firewall
```
firewall-cmd --permanent --zone=trusted --change-interface=docker0
firewall-cmd --permanent --zone=trusted --add-port=4243/tcp
firewall-cmd --reload
```

### Now you can reboot your node
```
reboot
```

**Once here, repeat the section preparing all nodes for node1 and node2.**

### Connect to your nodes cockpit interfaces
You should connect on root on all ur nodes.

* master [https://192.168.148.130:9090](https://192.168.148.130:9090)
* node1 [https://192.168.148.131:9090](https://192.168.148.131:9090)
* node2 [https://192.168.148.132:9090](https://192.168.148.132:9090)

### Deploying and starting Openshift Origin from master node
#### Preparation on master only
This declaration of targets for the sharing of the RSA key and the sending of keys which follow, simply avoids having to enter the login and password of each node when Ansible executes the installation scripts of Openshift Origin.

##### Creating an RSA key
```
ssh-keygen -q -N ""
```

##### Declare the target nodes for the key

```
vi ~/.ssh/config
```

```
Host master
    Hostname master.hal9000.com
    User root
Host node1
    Hostname node1.hal9000.com
    User root
Host node2
    Hostname node2.hal9000.com
    User root
```

##### Send the public-key to all the nodes
```
chmod 600 ~/.ssh/config
```

```
ssh-copy-id master
ssh-copy-id node1
ssh-copy-id node2
```

#### Preparing the hosts file for Ansible
```
vi /etc/ansible/hosts
```

```
#
# Ansible inventory for OpenShift Origin Platform  3.11
#

###########################################################################
#
# Ansible script that will be used to identify the targets of the deployment
# and use the variables for Openshift Origin
#
###########################################################################

###########################################################################
# Configuring your inventory file
# https://docs.openshift.com/container-platform/3.11/install/configuring_inventory_file.html
# add follows to the end
###########################################################################

###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
nodes
etcd

[masters]
master.hal9000.com openshift_schedulable=true containerized=false

[etcd]
master.hal9000.com

[nodes]
# Defined values for [openshift_node_group_name] in the file below
# [/usr/share/ansible/openshift-ansible/roles/openshift_facts/defaults/main.yml]
master.hal9000.com openshift_node_group_name='node-config-master-infra'
node1.hal9000.com openshift_node_group_name='node-config-compute'
node2.hal9000.com openshift_node_group_name='node-config-compute'

###########################################################################
### Ansible Vars
###########################################################################
[OSEv3:vars]
# Admin user created in previous section
ansible_ssh_user=root

# Use it if the installation user is not root
# ansible_become=true

# Deployment type
openshift_deployment_type=origin
# openshift_deployment_type=openshift-enterprise

# WARNING: only disable these checks in LAB/TEST environments(Do not use in production)
# The master node must have 16 Gb of memory and it works fine with 8 Gb
openshift_disable_check="disk_availability,memory_availability"

###########################################################################
### OpenShift Registries Locations
###########################################################################
# Use HTPasswd for authentication
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

###########################################################################
### OpenShift Master Vars
###########################################################################
# Define default sub-domain for Master node applications
openshift_master_default_subdomain=apps.hal9000.com

###########################################################################
# Image for a cluster with a working registry-console in 3.11
# Because the console which is installed by default has a bug that prevents it from starting
###########################################################################
openshift_cockpit_deployer_image='docker.io/timbordemann/cockpit-kubernetes:latest'

# Allow unencrypted connection within cluster
# Be careful, it's just to use a Cloud not connected to the internet, for a production target, you have to do it differently
openshift_docker_insecure_registries=192.168.1.16/16
```

#### Run prerequisites playbook
```
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml
```

#### Run deploy cluster playbook
```
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml
```

You have to get something like this.

```
INSTALLER STATUS ****************************************************************************************************************************************************************************************
Initialization               : Complete (0:00:33)
Health Check                 : Complete (0:02:18)
Node Bootstrap Preparation   : Complete (0:11:42)
etcd Install                 : Complete (0:00:55)
Master Install               : Complete (0:05:17)
Master Additional Install    : Complete (0:00:34)
Node Join                    : Complete (0:00:27)
Hosted Install               : Complete (0:00:53)
Cluster Monitoring Operator  : Complete (0:02:49)
Web Console Install          : Complete (0:01:35)
Console Install              : Complete (0:00:28)
metrics-server Install       : Complete (0:00:01)
Service Catalog Install      : Complete (0:04:50)
```

If after the execution you do not have an error message and you get a complete on all the steps, bravo! Openshift Origin is installed.

### Useful commands to verify that it works
#### See the state of your nodes
```
oc get nodes
```
You have to get something like this.

```
NAME                 STATUS    ROLES          AGE       VERSION
master.hal9000.com   Ready     infra,master   20m       v1.11.0+d4cacc0
node1.hal9000.com    Ready     compute        16m       v1.11.0+d4cacc0
node2.hal9000.com    Ready     compute        16m       v1.11.0+d4cacc0
```

#### View status with labels
```
oc get nodes --show-labels=true
```
You have to get something like this.

```
NAME                 STATUS    ROLES          AGE       VERSION           LABELS
master.hal9000.com   Ready     infra,master   20m       v1.11.0+d4cacc0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=master.hal9000.com,node-role.kubernetes.io/infra=true,node-role.kubernetes.io/master=true
node1.hal9000.com    Ready     compute        17m       v1.11.0+d4cacc0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node1.hal9000.com,node-role.kubernetes.io/compute=true
node2.hal9000.com    Ready     compute        17m       v1.11.0+d4cacc0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node2.hal9000.com,node-role.kubernetes.io/compute=true
```

#### See the state of your pods
```
oc get pods
```
You have to get something like this.

```
NAME                       READY     STATUS    RESTARTS   AGE
docker-registry-1-b528c    1/1       Running   0          43m
registry-console-1-frrb9   1/1       Running   0          43m
router-1-wz4bg             1/1       Running   0          43m
```

```
oc describe pod registry-console-1-frrb9
```
You have to get something like this.

```
Name:               registry-console-1-frrb9
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               master.hal9000.com/192.168.148.130
Start Time:         Thu, 01 Oct 2020 16:42:36 -0400
Labels:             deployment=registry-console-1
                    deploymentconfig=registry-console
                    name=registry-console
Annotations:        openshift.io/deployment-config.latest-version=1
                    openshift.io/deployment-config.name=registry-console
                    openshift.io/deployment.name=registry-console-1
                    openshift.io/scc=restricted
Status:             Running
IP:                 10.128.0.6
Controlled By:      ReplicationController/registry-console-1
Containers:
  registry-console:
    Container ID:   docker://5e6ff90a80d38a3a36981631e1e86025e329f9b7145c4c4ec469afbc05736a6f
    Image:          docker.io/timbordemann/cockpit-kubernetes:latest
    Image ID:       docker-pullable://docker.io/timbordemann/cockpit-kubernetes@sha256:f38c7b0d2b85989f058bf78c1759bec5b5d633f26651ea74753eac98f9e70c9b
    Port:           9090/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 01 Oct 2020 16:43:56 -0400
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:9090/ping delay=10s timeout=5s period=10s #success=1 #failure=3
    Readiness:      http-get http://:9090/ping delay=0s timeout=5s period=10s #success=1 #failure=3
    Environment:
      OPENSHIFT_OAUTH_PROVIDER_URL:  https://master.hal9000.com:8443
      OPENSHIFT_OAUTH_CLIENT_ID:     cockpit-oauth-client
      KUBERNETES_INSECURE:           false
      COCKPIT_KUBE_INSECURE:         false
      REGISTRY_ONLY:                 true
      REGISTRY_HOST:                 docker-registry-default.apps.hal9000.com
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mcsbs (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-mcsbs:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mcsbs
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  node-role.kubernetes.io/master=true
Tolerations:     <none>
Events:
  Type    Reason     Age   From                         Message
  ----    ------     ----  ----                         -------
  Normal  Scheduled  44m   default-scheduler            Successfully assigned default/registry-console-1-frrb9 to master.hal9000.com
  Normal  Pulling    44m   kubelet, master.hal9000.com  pulling image "docker.io/timbordemann/cockpit-kubernetes:latest"
  Normal  Pulled     42m   kubelet, master.hal9000.com  Successfully pulled image "docker.io/timbordemann/cockpit-kubernetes:latest"
  Normal  Created    42m   kubelet, master.hal9000.com  Created container
  Normal  Started    42m   kubelet, master.hal9000.com  Started container
```

The adresse of your OKD console is here : OPENSHIFT_OAUTH_PROVIDER_URL:  https://master.hal9000.com:8443

Because it's not possible to access to this address with the IP, you should open the hosts file of the computer you want to use to access to the OKD console and add the master DNS ident to the file.
```
192.168.1.16   master.hal9000.com  master
```

#### Create user accounts for OKD console
You can use the httpd-tools package to obtain the htpasswd binary that can generate these accounts.

```
yum -y install httpd-tools
```

##### Create a user account

```
touch /etc/origin/master/htpasswd
htpasswd -b /etc/origin/master/htpasswd admin redhat
```

You can create, for example, a user like this one, admin, with the password, redhat.

##### Restart OpenShift before going forward
```
master-restart api
master-restart controllers
```

Give this user account cluster-admin privileges, which allows it to do everything.

```
oc adm policy add-cluster-role-to-user cluster-admin admin
```

### Access the OKD console

[https://master.hal9000.com:8443](https://master.hal9000.com:8443)

![OKD Cockpit](images/OKD_console.jpeg)


<!-- ROADMAP -->
## Roadmap

See the [Project](https://github.com/AntoineMeheut/Rokc-Mk2/projects) for a list of proposed features (and known issues).

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create.
Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.

<!-- CONTACT -->
## Contact

If you want to contact me [just clic](mailto:github.contacts@protonmail.com)

Project Link: [https://github.com/AntoineMeheut/Rokc-Mk2](https://github.com/AntoineMeheut/Rokc-Mk2)

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
* [Installing OKD documentation](https://docs.okd.io/3.11/install/running_install.html)


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/AntoineMeheut/Rokc-Mk2?color=green
[contributors-url]: https://github.com/AntoineMeheut/Rokc-Mk2/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/AntoineMeheut/Rokc-Mk2
[forks-url]: https://github.com/AntoineMeheut/Rokc-Mk2/network/members
[stars-shield]: https://img.shields.io/github/stars/AntoineMeheut/Rokc-Mk2
[stars-url]: https://github.com/AntoineMeheut/Rokc-Mk2/stargazers
[issues-shield]: https://img.shields.io/github/issues/AntoineMeheut/Rokc-Mk2
[issues-url]: https://github.com/AntoineMeheut/Rokc-Mk2/issues
[license-shield]: https://img.shields.io/github/license/AntoineMeheut/kafka_producer
[license-url]: https://github.com/AntoineMeheut/kafka_producer/blob/master/LICENSE
