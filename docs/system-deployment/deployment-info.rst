=========================================
System Deployment Info
=========================================

The following user guide outlines the steps that will be done in order to deploy ovn-domain-service.

Control-Plane Deployment
----------------------------

**Prerequisites**

for inventory file variables setup refer to `TODO add link <https://gitlab-master.nvidia.com/sdn/ovn-isolation-deployment/-/blob/main/testing/control-plane/README.md?ref_type=heads>`__


**Deployment**

This deployment will orchestrate three containers using Docker, running them as a systemd service:

- OVN Central Service

    Provides centralized control plane for OVN

- PostgreSQL Service

    Stores service state data


- Domain Isolation Service

    Depends on OVN Central and PostgreSQL containers, contains the gRPC server of the ovn isolation service API


Prior to deployment, the following dependencies will be installed on the remote machine:

- Python Package Manager (pip3)

- Docker Engine

The deployment process will copy the three service files, start the services, and enable them to run automatically. This will launch the Docker containers, establish persistent volumes, create necessary links and dependencies between the containers, and apply any additional configurations required.


Compute Node Deployment (x86 Host)
-------------------------------------

**Prerequisites**

for inventory file variables setup refer to `TODO add link <https://gitlab-master.nvidia.com/sdn/ovn-isolation-deployment/-/blob/main/testing/dpu/inventory?ref_type=heads>`__


**Deployment**

As part of this deployment, Virtual Functions (VFs) will be created to enable SR-IOV (Single Root I/O Virtualization) capabilities. The number of VFs to be created is specified by the `num_vfs` variable defined in the inventory file.


DPU node deployment
----------------------

**Prerequisites**

for inventory file variables setup refer to `TODO add link <https://gitlab-master.nvidia.com/sdn/ovn-isolation-deployment/-/blob/main/testing/dpu/inventory?ref_type=heads>`__


**Deployment**

This deployment will configure Open vSwitch (OVS) to be based on DOCA using DPDK (Data Plane Development Kit) and enabling hardware offload capabilities. Ensure that the FLEX_PARSER_ENABLE parameter is set to 8 to enable flexible parser functionality. Additionally, it will:

- Set the Number of Huge Pages

    Configure the appropriate number of huge pages required for DPDK operations.

- Set OVN Bridge Datapath Type

    Set the OVN bridge datapath type to netdev for improved performance.

- Create OVS Bridge and Connect Uplink

    Create an OVS bridge and connect the uplink interface to it for external connectivity.

- Set Encapsulation IP Addresses for Tunneling

    Define the encapsulation IP addresses to be used for tunneling traffic between OVS instances.

- Start OVN Controller on DPUs

    Deploy the OVN controller on the DPUs as a Docker container, which will create the `br-int` integration bridge.

- Bind Interface Representors (Optional)

    Optionally, bind interface representors to the `br-int` bridge for direct communication between VMs and the physical network.

    This comprehensive configuration ensures optimal performance, hardware offload support, and seamless integration of OVS with the OVN controller for software-defined networking capabilities.
