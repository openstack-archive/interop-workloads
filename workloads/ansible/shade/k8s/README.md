> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
>
>   http://www.apache.org/licenses/LICENSE-2.0
>
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an "AS IS" BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
> See the License for the specific language governing permissions and
> limitations under the License.

# Kubernetes Ansible deployments on OpenStack Cloud

This ansible playbook will install a 3 node kubernetes cluster. The first
node will be used as the master node, rest of the nodes will be used as
kubernetes worker node.

Once the script finishes, a kubernetes cluster should be ready for use.

## Status

In process

## Requirements

- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Install openstack shade] (http://docs.openstack.org/infra/shade/installation.html)
- Make sure there is an Ubuntu cloud image available on your cloud.
- Clone this project into a directory.


If you will be using an Ubuntu system as Ansible controller, then you can
easily setup an environment by running the following script. If you have
other system as your Ansible controller, you can do similar steps to setup
the environment, the command may not be exact the same but the steps you
need to do should be identical.

    sudo apt-get update

    sudo apt-get install python-dev python-pip libssl-dev libffi-dev -y
    sudo pip install --upgrade pip

    sudo pip install six==1.10.0
    sudo pip install shade==1.16.0
    sudo pip install ansible==2.2.1.0
    sudo ansible-galaxy install vmware.coreos-bootstrap

    git clone https://github.com/openstack/interop-workloads.git

This workload requires that you use Ansible version 2.2.0.0 or above due to
floating IP allocation upgrades in Ansible OpenStack cloud modules.

### Prep

#### Deal with ssh keys for Openstack Authentication

If you do not have a ssh key, then you should create one by using a tool.
An example command to do that is provided below. Once you have a key pair,
ensure your local ssh-agent is running and your ssh key has been added.
This step is required. Not doing this, you will have to manually give
passphrase when script runs, and script can fail. If you really do not want
to deal with passphrase, you can create a key pair without passphrase::

    ssh-keygen -t rsa -f ~/.ssh/interop
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/interop

#### General Openstack Settings

Ansible's OpenStack cloud module is used to provision compute resources
against an OpenStack cloud. Before you run the script, the cloud environment
will have to be specified. Sample files have been provided in vars directory.
If you target ubuntu, you should use vars/ubuntu.yml as the sample, if you
target coreos, you should use vars/coreos.yml file as the sample to create
your own environment file. Here is an example of the file::

    auth: {
      auth_url: "http://x.x.x.x:5000/v3",
      username: "demo",
      password: "{{ password }}",
      domain_name: "default",
      project_name: "demo"
    }

    app_env: {
      target_os: "ubuntu",
      image_name: "ubuntu-16.04",
      region_name: "RegionOne",
      availability_zone: "nova",
      validate_certs: True,
      private_net_name: "my_tenant_net",
      flavor_name: "m1.medium",
      public_key_file: "/home/ubuntu/.ssh/interop.pub",
      private_key_file: "/home/ubuntu/.ssh/interop"
      stack_size: 3,
      volume_size: 2,
      block_device_name: "/dev/vdb",

      domain: "cluster.local",
      pod_network: {
        Network: "172.17.0.0/16",
        SubnetLen: 24,
        SubnetMin: "172.17.0.0",
        SubnetMax: "172.17.255.0",
        Backend: {
          Type: "udp",
          Port: 8285
        }
      },
      service_ip_range: "172.16.0.0/24",
      dns_service_ip: "172.16.0.4",

      flannel_repo: "https://github.com/coreos/flannel/releases/download/v0.7.0/flannel-v0.7.0-linux-amd64.tar.gz",
      k8s_repo: "https://storage.googleapis.com/kubernetes-release/release/v1.5.3/bin/linux/amd64/"
    }

The values of the auth section should be provided by your cloud provider. When
use keystone 2.0 API, you will not need to setup domain name. You can leave
region_name empty if you have just one region. You can also leave
private_net_name empty if your cloud does not support tenant network or you
only have one tenant network. The private_net_name is only needed when you
have multiple tenant networks. validate_certs should be normally set to True
when your cloud uses tls(ssl) and your cloud is not using self signed
certificate. If your cloud is using self signed certificate, then the
certificate can not be easily validated by ansible. You can skip it by setting
the parameter to False. currently the only value available for target_os is
ubuntu and coreos. Supported ubuntu releases are 16.04 and 16.10. Supported
coreos is the stable coreos openstack image.

You should use a network for your OpenStack VMs which will be able to access
internet. For example, in the example above, the parameter private_net_name
was configured as my_tenant_net, this will be a network that all your VMs
will be connected on and the network should have been connected with a router
which routes traffic to external network.

stack_size is set to 3 in the example configuration file, you can change that
to any number you wish, but it must be 2 at minumum. In that case, you will
have one master node and one worker node for k8s cluster. If you set stack_size
to a bigger number, one node will be used as the master, and the rest of the
nodes will be used as worker. Please note that master node will also act as
worker node.

public key and private key files should be created before you run the workload
these keys can be located in any directory that you prefer with read access.

volume_size and block_device_name are the parameter that you can set to allow
the workload script to provision the right size of cinder volume to create
k8s volumes. Each cinder volume will be created, partitioned, formated, and
mounted on each worker and master node. The mount point is /storage. A pod or
service should use hostPath to use the volume.

The workload is currently developed using flannel udp for k8s networking.
Other networking configurations can be used by simply changing the value of
flannel_backend parameter, but before you change the values, you will have to
make sure that the underlying networking is configured correctly.

The flanned_repo and k8s_repo point to the offical repositories of each
component, you may choose to setup a local repository to avoid long
download time especially when your cloud is very remote to these offical
repository. To do that, you only need to setup a http server and place the
following binaries in your http server directory.

    - kubelet
    - kubectl
    - kube-proxy
    - kube-apiserver
    - kube-controller-manager
    - kube-scheduler
    - flannel-v0.7.0-linux-amd64.tar.gz

## Run the script to create a kubernetes cluster using coreos image

Coreos images does not have python installed and it needs to be bootstraped.
To do that, you will have to install a bootstrap on your ansible controller
first by executing the following command, this only needs to be done once. We
simply use vmware coreos bootstrap, you can choose other ones, but this is
the one we have been using for testings.

    ansible-galaxy install vmware.coreos-bootstrap

With your cloud environment set, you should be able to run the script::

    ansible-playbook -e "action=apply env=coreos password=XXXXX" site.yml

The above command will stand up a kubernetes cluster at the environment
defined in vars/coreos.yml file. Replace xxxxx with your own password.

## Run the script to create a kubernetes cluster using ubuntu image

With your cloud environment set, you should be able to run the script::

    ansible-playbook -e "action=apply env=ubuntu password=XXXXX" site.yml

The above command will stand up a kubernetes cluster at the environment
defined in vars/ubuntu.yml file. Replace xxxxx with your own password.


## The results of the work load successful run

If everything goes well, it will accomplish the following::

    1. Provision 3 nodes or the number of nodes configured by stack_size
    2. Create security group
    3. Add security rules to allow ping, ssh, and kubernetes ports
    4. Install common software onto each node such as docker
    5. Download all the required software onto the master node
    6. Setup the master node with kube-apiserver, kube-controller-manager and
       kube-scheduler and configur each kubernetes services on the master node
    7. Download software for worker node from the master node.
    8. Setup flanneld, docker, kubelet and kube-proxy on each work node.
    9. Install kubernetes dashboard and dns services.


## The method to run just a play, not the entire playbook

The script will create an ansible inventory file name runhosts at the very
first play, the inventory file will be place at the run directory of the
playbook root. If you like to only run specify plays, you will be able to run
the playbook like the following:

    ansible-playbook -i run/runhosts -e "action=apply env=leap password=XXXXX" site.yml
    --tags "common,master"

The above command will use the runhosts inventory file and only run plays
named common and master, all other plays in the play book will be skipped.


## Next Steps

### Check its up

If there are no errors, you can use kubectl to work with your kubernetes
cluster.

## Cleanup

Once you're done with it, don't forget to nuke the whole thing::

    ansible-playbook -e "action=destroy env=leap password=XXXXX" site.yml

The above command will destroy all the resources created.
