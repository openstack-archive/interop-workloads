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
Telco needs to leverage a MANO to deploy corresponsding network services
along with underlying infrastructure.
In this blueprint, we deploy the voice calling workload (vIMS and Open-O,
which serves as VNF and MANO respectively) on an OpenStack infrstructure.

Network Function Virtualization(NFV) is a network architecture concept that
uses virtualization technology in Telco industry. Virtual Network Function
(VNF) is a software implementation of network functions that can be deployed
on a Network Virtualization Infrastructure(NFVI). OpenStack is a type of
NFVI.

Running a VNF on a NFVI is a normal way to demonstrates the ability that
OpenStack supports NFV. To fulfill the goal, a MANO is needed to manage the
lifecycle of VNF and orchestrate the services.
OPEN-O is an open source MANO project.

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

  * Shell script to deploy a SIP client

* Test scripts to verify the workload

  * Python script to confirm OPEN-O is working

  * Ruby script to confirm vIMS is working


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

1. OPEN-O VM
  * at least 4 CPUs, 64G RAM, 100G Disk
  * Ubuntu 16.04
2. vIMS VM * 7 nodes
  * 1 CPU, 2G RAM, 2G RAM, 20G Disk
  * Ubuntu 12.04
3. OpenStack Keystone V2 is verified while Keystone V3 is not verified


Details

1. Deployment scripts development to fulfill the workload

  The workload of running vIMS on OpenStack with OPEN-O

  1. Deploy 1 VM by OpenStack and install the OPEN-O
    * refer to the prerequisite for the VM requirement

  2. Bind the OPEN-O with OpenStack

    * configure OpenStack as Virtual Infrastructure Manager(VIM) by calling
    OPEN-O API, which is mainly about providing OpenStack authentication
    information to OPEN-O

  3. Deploy the vIMS by OPEN-O

    * define the Network Service Descriptor(NSD) and VNF Descriptor(VNFD) to
    give the overall definition for the topology

    * deploy the topology by OPEN-O

      * several VMs are deployed to play different roles. A Clearwater vIMS is
      consist of 7 VMs includes basic function nodes and a DNS.
      * refer to the prerequisite for the VM requirement

  4. Configure vIMS and get specific calling number for each OpenStack vendor

    * call vIMS API to generate identification for each OpenStack vendor

  5. Configure the SIP client with the calling identification

    * call the SIP client API to configure

  6. Show the audiences by dialing a specific number

2. Test scripts to verify the deployment

  * script to confirm OpenStack is working

    * basic scenario to create VM along with network as API verification for
    the OpenStack

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
