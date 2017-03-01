# NFV Ansible deployments on OpenStack Cloud

This ansible playbook will install an OPEN-O first then deploy Clearwater vIMS by OPEN-O.

Once the script finishes, a vIMS is ready to be used.

## Status

In process

## Requirements

- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Install docker-py](https://github.com/docker/docker-py)
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
Execute prepare_env.sh to install Ansible and other required packages

#### General OpenStack Settings

Before running the script, cloud environment authentication needs to be provided.
Place your authentication file as files/admin-openrc.sh. 

There are two examples for your reference

  ##### Keystone V3: files/admin-openrc-v3-sample.sh::

    export OS_PROJECT_DOMAIN_NAME=default
    export OS_USER_DOMAIN_NAME=default
    export OS_PROJECT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=password
    export OS_AUTH_URL=http://172.16.1.222:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_REGION_NAME=RegionOne

  ##### Keystone V2.0: files/admin-openrc-v2-sample.sh::

    export OS_PASSWORD=password
    export OS_TENANT_NAME=admin
    export OS_AUTH_URL=http://172.16.1.222:5000/v2.0
    export OS_USERNAME=admin
    export OS_REGION_NAME=RegionOne

Make sure every field in the sample is supplied.


## Run the script to deploy vIMS with the help of OPEN-O

With your cloud environment set, you should be able to run the script::

    ansible-playbook nfv_launch.yml

The above command will firstly deploy OPEN-O, then a vIMS will be set up by OPEN-O.

Also, OPNFV Functest is leveraged to test the vIMS to make sure it is working.


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


## Next Steps

### Make a call via SIP client

Install the SIP client, e.g. jitsi and configure the client with calling information

### Cleanup

Script is needed to clean up the environment.
