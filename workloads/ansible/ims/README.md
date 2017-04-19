# IMS Ansible deployments on OpenStack Cloud

## Status

This will install 7 nodes for clearwater.
  ellis: provisioning portal
  bono: Edge Proxy
  sprout: SIP Router
  homer: XDMS
  homestead: HSS Cache
  ralf: CTF
  dns_server: DNS Server


Once the script finishes, you can use sip client connect it and make a call.

## Requirements

- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Install openstack shade] (http://docs.openstack.org/infra/shade/installation.html)
- Make sure there is an Ubuntu cloud image(14.04) available on your cloud.
- Clone this project into a directory.

## Ansible

Ansible and OpenStack Shade will be used to provision all of the OpenStack
resources required by this workload.

### Prep

#### Deal with ssh keys for Openstack Authentication

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
      image_name: "ubuntu-14.04",
      region_name: "RegionOne",
      availability_zone: "nova",
      validate_certs: False,
      ssh_user: "ubuntu",
      private_net_name: "my_tenant_net",
      public_key_file: "/home/ubuntu/.ssh/id_rsa.pub",
      flavor_name: "m1.small",
      clearwater_repo: "deb http://repo.cw-ngv.com/archive/repo117 binary/",
      clearwater_key: "http://repo.cw-ngv.com/repo_key"
    }


## Provision the IMS nodes

With your cloud environment set, you should be able to run the script::

    ansible-playbook -e "action=apply env=config password=XXXXX" site.yml

The command will stand up the nodes using a cloud named leconfigap
(vars/config.yml).
If you run the test against other cloud, you can create a new file use same
structure and specify that cloud attributes such as auth_url, etc. Then you
can simply replace work leap with that file name. Replace xxxxx with your
own password.

If everything goes well, it will accomplish the following::

    1. Provision 7 nodes
    2. Create security group
    3. Add security rules to allow ping, ssh, mysql and sip access
    4. Set up clearwater on these nodes


## Next Steps

### Check its up

If there are no errors, you can use tow sip clients connect it, such as Zoiper.
Firstly, you need login ellis and create some phone numbers. And add the phone
number and the sip client.
Then you can make call with the sip client.

## Cleanup

Once you're done with it, don't forget to nuke the whole thing::

    ansible-playbook -e "action=destroy env=config password=XXXXX" site.yml

The above command will destroy all the resources created.
