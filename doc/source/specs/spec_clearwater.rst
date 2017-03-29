
======================================
 Deploy IMS(Clearwater) on OpenStack
======================================

IMS(the IP Multimedia Subsystem) is a typical scenario in telecommunication
industry. This workload consists of VNF functionality, and can make the
voice calling as a NFV workload running atop OpenStack Infrastructure.
In this blueprint, we deploy the voice calling workload (Clearwater) on the
OpenStack infrstructure.

Clearwater is an open source implementation of IMS(the IP Multimedia Subsystem)
designed from the ground up for massively scalable deployment in the Cloud to
provide voice, video and messaging services.

Problem description
===================

There is no existing IMS workload running on OpenStack to test interoperability.


Proposed change
===============

The primary changes that need to be done are as follows:

* Ansible script to deploy IMS(Clearwater)

* Verify the workload with ruby script


Implementation
==============

Assignee(s)
-----------

Primary assignee:

Other contributors:

Milestones
----------

Target Milestone for completion:
  Pike-1

Work Items
----------

Prerequisites

1. IMS VM * 7 nodes
  * 1 CPU, 2G RAM, 2G RAM, 20G Disk
  * Ubuntu 14.04
2. SIP client


Details

1. Ansible script to deploy IMS(Clearwater)

  * 7 VMs are deployed to play different roles:
  ellis, bono, sprout, homer, homestead, ralf and dns_server.
  * refer to the prerequisite for the VM requirement

2. Verify the workload with ruby script

  * Call Clearwater's test code to confirm main functions are working

Dependencies
============

- Include specific references to specs and/or blueprints in interop-workloads-specs, or in other
  projects, that this one either depends on or is related to.

  None

- Does this feature require any new library dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library?

  Clearwater IMS, SIP client
