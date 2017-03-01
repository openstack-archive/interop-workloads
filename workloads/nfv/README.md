# NFV Ansible deployments on OpenStack Cloud

This ansible playbook will install an OPEN-O first then deploy
Clearwater vIMS by OPEN-O.

Once the script finishes, a vIMS is ready to be used.

## Status

In process

## Requirements

- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Install docker-py 1.9.0](https://github.com/docker/docker-py)
- Make sure there is a VM to be used to install OPEN-O.

  It should meet the requirement below

  - at least 4 CPUs, 64G RAM, 100G Disk

- Make sure the OpenStack cloud is able to deploy at least 7 nodes.

  Each node requires resources as below

  - 1 CPU, 2G RAM, 2G RAM, 20G Disk

- Clone this project into a directory to the VM

## Ansible

Ansible will be used to orchestrator the whole process

### Prep

#### Running environment Setup
Execute tox to install Ansible and other required packages

#### General OpenStack Settings

Before running the script, cloud environment authentication needs to be
provided. Please update the file under group_vars/all

There is one example for your reference

  ##### group_vars/all::

    os_project_domain_name: default
    os_user_domain_name: default
    os_username: admin
    os_password: password
    os_project_name: admin
    os_auth_url: http://192.168.136.222:5000/v3
    os_identity_api_version: 3
    os_region_name: RegionOne

## Run the script to deploy vIMS with the help of OPEN-O

With your cloud environment set, you should be able to run the script::

    ansible-playbook nfv_launch.yml

The above command will firstly deploy OPEN-O, then a vIMS will be set up by
OPEN-O.

Also, OPNFV Functest is leveraged to test the vIMS to make sure it is
working.


## The results of the work load successful run

If everything goes well, it will accomplish the following::

    1. Install OPEN-O onto docker container
    2. Create security group
    3. Add security rules to allow icmp, tcp and ucp ports
    4. Deploy a VM on OpenStack and install Juju
    5. Connect Juju with OPEN-O and OpenStack
    6. Configure OPEN-O with vIMS topology definition
    7. Deploy vIMS Clearwater via OPEN-O
    8. Test the vIMS Clearwater by leveraging OPNFV Functest
    9. Create user on Ellis of vIMS Clearwater
    10. Create calling number on Ellis of vIMS Clearwater

### Environment Information

- OPEN-O can be accessed through

http://VM_host_IP/openoui/common/default.html

- Calling information is put under /home/opera/ellis.info

- Log is put under /home/opera/opera.log


## Execution Duration

The process involves pulling 20+ docker images and downloading about nearly 4G
OpenStack images. The time it takes can be impacted by the network enormously.
Thus the duration varies from lab to lab. If this step is once run, it will be
skipped in the future round of execution. One suggestion is to run this step
in advance to decrease the whole execution duration.

The deployment of OPEN-O takes approximately 15~20 minutes.

The deployment of vIMS Clearwater takes about 20 minutes. It also depends on
the network as vIMS downloads quite a number of packages.

## Next Steps

### Make a call via SIP client

Install the SIP client, e.g. jitsi and configure the client with calling
information

### Cleanup

Script is needed to clean up the environment.
