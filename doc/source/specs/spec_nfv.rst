..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/interop-workloads-specs/+spec/awesome-thing should be named
  awesome-thing.rst .  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None
  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==================================
 Run vIMS on OpenStack with OPEN-O
==================================

Network Function Virtualization(NFV) is a network architecture concept that
uses virtualization technology in Telco industry. Virtual Network Function
(VNF) is a software implementation of network functions that can be deployed
on a Network Virtualization Infrastructure(NFVI). OpenStack is a type of
NFVI.

Running a VNF on a NFVI is a typical way to demonstrates the ability that
OpenStack supports NFV. To fulfill the goal, a NFV Management & Orchestration
(MANO) is needed to manage the lifecycle of VNF and orchestrate the services.
OPEN-O is an open source MANO project.

Virtual IP Multimedia Subsystem(vIMS) network is a core component of deploying
VoLTE services in an LTE network and it a good candidate for showing the
OpenStack deployment ability. Clearwater is an open source implementation of
vIMS.


Problem description
===================

There is no existing NFV workload running on OpenStack to test interoperability.


Proposed change
===============

The primary changes that need to be done are as follows:

* Deployment scripts to deploy the workload
* Test scripts to verify the workload


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

1. Deployment scripts development to fulfill the workload

  The workload of running vIMS on OpenStack with OPEN-O

  1. Deploy 1 VM by OpenStack and install the OPEN-O

  2. Bind the OPEN-O with OpenStack

  3. Deploy the vIMS by OPEN-O

  4. Configure vIMS and set specific calling number for each OpenStack vendor

  5. Show the audiences by dialing a specific number
   What type of SIP Client will be used for the showcase?
2. Test scripts to verify the deployment


Dependencies
============

- Include specific references to specs and/or blueprints in interop-workloads-specs, or in other
  projects, that this one either depends on or is related to.

  None

- Does this feature require any new library dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library?

  OPEN-O, Clearwater vIMS
