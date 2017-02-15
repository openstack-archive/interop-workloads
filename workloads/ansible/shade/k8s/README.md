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

## Ansible

Ansible and OpenStack Shade will be used to provision all of the OpenStack
resources required by kubernetes.

### Prep

#### Deal with ssh keys for Openstack Authentication

If you do not have a ssh key, then you should create one by using a tool.
An example command to do that is provided below. Once you have a key pair,
ensure your local ssh-agent is running and your ssh key has been added.
This step is required. Not doing this, you will have to manually give
passphrase when script runs, and script can fail. If you really do not want
to deal with passphrase, you can create a key pair without passphrase::

    ssh-keygen -t rsa
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_rsa

#### General Openstack Settings

Ansible's OpenStack cloud module is used to provision compute resources
against an OpenStack cloud. Before you run the script, the cloud environment
will have to be specified. Sample files have been provided in vars directory.
You may create one such file per cloud for your tests.

    auth: {
      auth_url: "http://x.x.x.x:5000/v3",
      username: "demo",
      password: "{{ password }}",
      domain_name: "default",
      project_name: "demo"
    }

    app_env: {
      target_os: "ubuntu",
      image_name: "ubuntu-15.04",
      region_name: "RegionOne",
      availability_zone: "nova",
      validate_certs: True,
      private_net_name: "my_tenant_net",
      flavor_name: "m1.medium",
      public_key_file: "/home/ubuntu/.ssh/interop.pub",
      private_key_file: "/home/ubuntu/.ssh/interop"
      stack_size: 3,
      flannel_repo: "https://github.com/coreos/flannel/releases/download/v0.7.0/flannel-v0.7.0-linux-amd64.tar.gz",
      etcd_repo: "https://github.com/coreos/etcd/releases/download/v2.2.5/etcd-v2.2.5-linux-amd64.tar.gz",
      k8s_repo: "https://storage.googleapis.com/kubernetes-release/release/v1.5.0/bin/linux/amd64/"
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
ubuntu. Supported ubuntu releases are 16.04 and 16.10.


## Run the script to create a kubernetes cluster

With your cloud environment set, you should be able to run the script::

    ansible-playbook -e "action=apply env=leap password=XXXXX" site.yml

The command will stand up the nodes using a cloud named leap (vars/leap.yml).
If you run the test against other cloud, you can create a new file use same
structure and specify that cloud attributes such as auth_url, etc. Then you
can simply replace work leap with that file name. Replace xxxxx with your
own password.

If everything goes well, it will accomplish the following::

    1. Provision 3 nodes or the number of nodes configured by stack_size
    2. Create security group
    3. Add security rules to allow ping, ssh, and kubernetes ports
    4. Install common software onto each node such as docker
    4. Download all the required software onto the master node
    5. Setup the master node with kube-apiserver, kube-controller-manager and
       kube-scheduler and configur each kubernetes services on the master node
    6. Download software for worker node from the master node.
    7. Setup flanneld, docker, kubelet and kube-proxy on each work node.

The script will create an ansible inventory file name runhosts at the very
first play, the inventory file will be place at the home directory of the
ansible user. If you like to only run specify plays, you will be able to run
the playbook like the following:

    ansible-playbook -i ~/runhosts -e "action=apply env=leap password=XXXXX" site.yml 
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
