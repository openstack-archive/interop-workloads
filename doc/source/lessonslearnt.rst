Overview
--------

In the first round of interop challenge testing, although no comprehensive evaluations
were performed and there may be other good options, we had the following findings
for the workload providers:

* For interoperable automated deployment, Ansible + Ansible OpenStack cloud
modules (based on OpenStack Shade) provided the best results in our tests.
* It is recommended to be prepared to include project network configuration as
part of the workload
* It is recommended to structure your workload so that it can adapt to either
attach instances to a routeable network or use floating IP's based on the given
parameters
* It is recommended to parameterize things that are likely to change across
different cloud/guest OS setups
* It is recommended to allow the user to set the network names (like eth0) as
parameters to the workload or try to detect these names in the workload when the
network nic is needed
* It is recommended to check if the cloud supports cloud-init given that workloads
heavily relying on metadata service might fail on clouds that don't have metadata
support

Detailed explanations and examples could be found in the following sections.

Tooling
-------

For interoperable automated deployment, Ansible + Ansible OpenStack cloud
modules (based on OpenStack Shade) provided the best results.

Terraform and its OpenStack cloud modules (based on OpenStack Shade) has
also been tried, but various issues such as not supporting multiple same
service endpoints renders many clouds which support multiple endpoints for
Nova, Neutron versions rendered failed deployment on these clouds, but
supporting multiple endpoints is necessary for different versions of
OpenStack client applications. Terraform also does not allow apply (creation)
and destroy (remove) action to be used at the same time. But it is often
needed, for example, during a deployment, you may need a floating IP (that is
the apply action in Terraform) but at the end of the deployment you may want
to remove that floating IP (that is the destroy action in Terraform), so the
floating IP which is a resource in Terraform only existed in a short period
of time, Terraform can not really handle the situation. This is probably the
most unforgiven restriction of the Terraform. The Interop Challenge working
group can not seem find a work around to overcome the restriction.  Also it
appears that these issues have been identified but have not been actively
addressed by Terraform community.

OpenStack Heat has also been discussed but since the adoption of HEAT is
still not wide spread, this tool was not used. Similar reasons for other
tools like Murano and Juju.

It's perhaps worth noting that both the Ansible OpenStack cloud modules and
Terraform OpenStack cloud modules based on OpenStack Shade, which is
a library that was written explicitlly to work around some Interop
problems. So we can essentially have some degree of interop as long as
there is an interop layer between us and the cloud (the aim should be not
to need this library), tooling in interop challenge is a very important
subject.

Shade seems to be missing AZ parameter for create_keypair (Ansible's
os_keypair) and other functions which can cause problems on clouds with
multi-AZs per region.


Networking
----------

Network virtualization features are where most interoperability issues become
visible. OpenStack Neutron support very large number of plugins, these plugins
can behave very differently. For example, private IP and floating IP
supporting can vary, some clouds make public accessable IP address as private
IP address when returned from client library, some clouds make the same thing
as public IP address, the later seems to be the right behavior, but clouds
implement them differently. Layer 2 and layer 3 functions can be also
challenge, some clouds won't expose the functions for customers to create
routers, or networks. Releasing the alocated floating IPs is completely
missing from all OpenStack cloud modules tools like Ansible and Terraform.
This problem results in the alocated floating IPs hanging around, it is
especially bad for clouds which do not have small public IP address segment.

Not all clouds provide tenant networks by default.  Be prepared to configure
your own tenant network if the cloud supports tenant network.

Can not assume the first NIC on the guest is going to be eth0 (this is common
on older guest OS's prior to the arrival of Predictable Network Interface
Names and systemd, and likely isn't true on newer guest OS's). Instead, allow
the user to set those as parameters to the workload or try to detect these
names in the workload when the network nic is needed.

Not all clouds support floating IP or private IP. You may want to structure
your workload so that it can adapt to either attach instances to a routeable
network or use floating IP's based on the parameters it's given.

The tenant network has its advantages when the communications are server to
server on the same network. For example, when your deployment scenario
involves multiple backend servers such as database and application servers,
the commuincation between these servers can be placed on the tenant network
to improve security and performance.


Provisiong
----------

It makes a real difference not only the HW that the cloud is running on but
also if the backend is ceph or something else, if it is co-located, if the
images have any sort of overhead checks, etc.

If you don't assume a particular guest OS image, be careful with
storage/networking.  We encountered one example in which a particular
guestOS/virtual adapter pair needed to rescan the SCSI bus before it would
recognize a newly attached Cinder volume. Rescanning the bus is generally
harmless if not needed and ensures that images built with adapter types that
need it run successfully, so it's an example of something you can do to make
your workloads more interoperable.

Parameterize things that are likely to change across different cloud/guest
OS setups.  For example: don't assume the first volume attached to a guest
will always be /dev/vdb (this is common but not guaranteed on libvirt, often
untrue on other hypervisors).


Metadata
--------

Not all cloud support cloud-init, when develop workloads which heavily rely
on metadata services, the clouds without metadata support will fail.


Conclusion
----------

With best practices it is possible to create enterprise applications (with
enterprise characteristics such as load balancer, multiple web application
servers, distributed database, security groups, block storage to provide
enterprise level networking safeguards) can be created such that they are
portable to numerous (over 18) private and public OpenStack Clouds.
