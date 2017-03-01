..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..

==================================
 Run vIMS on OpenStack with OPEN-O
==================================

Making a voice call is a typical scenario in telecommunication industry.
To make the voice calling as a NFV workload running atop OpenStack
Infrastructure, such workload consists of VNF and MANO functionality,
Telco needs to leverage a MANO to deploy corresponding network services
along with underlying infrastructure.
In this blueprint, we deploy the voice calling workload (vIMS and Open-O,
which serves as VNF and MANO respectively) on an OpenStack infrastructure.

Network Function Virtualization(NFV) is a network architecture concept that
uses virtualization technology in Telco industry. Virtual Network Function
(VNF) is a software implementation of network functions that can be deployed
on a Network Virtualization Infrastructure(NFVI). Virtualized Infrastructure
Manager(VIM) is responsible for controlling and managing the NFVI resources.
OpenStack is a VIM.

Running a VNF on a NFVI with the help of VIM is a normal way to demonstrate
the ability that OpenStack supports NFV. To fulfill the goal, a MANO is needed
to manage the life-cycle of VNF and orchestrate the services.
OPEN-O is an open source MANO project.
https://www.open-o.org/

vIMS network is a core component of deploying VoLTE services in an LTE network
and it a good candidate for showing the OpenStack deployment ability.
Clearwater is an open source implementation of vIMS.


Problem description
===================

There is no existing NFV workload running on OpenStack to test interoperability.


Proposed change
===============

The primary changes that need to be done are as follows:

* Deployment scripts to deploy the workload

  * Shell script to deploy OPEN-O

  * Python script to call OPEN-O to deploy vIMS

* Test scripts to verify the workload

  * Python script to confirm OPEN-O is working

  * Python script to confirm vIMS is working


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Helan Yao <yaohelan@huawei.com>

Other contributors:
  Zhipeng Huang <huangzhipeng@huawei.com>

Milestones
----------

Target Milestone for completion:
  Pike-1

Work Items
----------

Prerequisites

1. VM to host OPEN-O containers
  * at least 4 CPUs, 64G RAM, 100G Disk
  * Ubuntu 14.04
2. vIMS VM * 7 nodes
  * 1 CPU, 2G RAM, 2G RAM, 20G Disk
  * Ubuntu 12.04
3. OpenStack Keystone V2 and V3 are supported
4. SIP client to make the call


Details

1. Deployment scripts development to fulfill the workload

  The workload of running vIMS on OpenStack with OPEN-O

  1. Deploy OPEN-O docker containers on the host

    * refer to the prerequisite for the VM requirement

  2. Create security group and add security rules to allow icmp, tcp and ucp ports

  3. Deploy a VM on OpenStack and install Juju

  4. Connect Juju with OPEN-O

    * configure Juju to OPEN-O as VNFM

  5. Connect OPEN-O with OpenStack

    * configure OPEN-O to use OpenStack as its VIM. This can be done by providing
    the OpenStack authentication information to the orchestrator.

  6. Deploy the vIMS by OPEN-O

    * define the Network Service Descriptor(NSD) and VNF Descriptor(VNFD) to
    give the overall definition for the topology

    * deploy the topology by OPEN-O

      * several VMs are deployed to play different roles. A detailed architecture of
      Clearwater vIMS can be referred here
      http://www.projectclearwater.org/technical/clearwater-architecture

      * refer to the prerequisite for the VM requirement

  7. Configure vIMS and get specific calling number for each OpenStack vendor

    * call vIMS API to generate identification for each OpenStack vendor

  8. Configure the SIP client with the calling identification

    * call the SIP client API to configure

  9. Show the audiences by dialing a specific number

2. Test scripts to verify the deployment

  OPNFV Functest project is leveraged to verify the deployment in terms of

  * script to confirm OpenStack is working

    * basic scenario to confirm OpenStack is healthy

  * script to confirm OPEN-O is working

    * basic scenario to call OPEN-O services to confirm core services are working

  * script to confirm vIMS is working

    * basic scenario to call vIMS services to confirm main functions are working

Dependencies
============

- Include specific references to specs and/or blueprints in interop-workloads-specs, or in other
  projects, that this one either depends on or is related to.

  None

- Does this feature require any new library dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library?

  OPEN-O, Clearwater vIMS, SIP client
