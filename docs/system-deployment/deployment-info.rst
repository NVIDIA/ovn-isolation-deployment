=========================================
System Deployment Info
=========================================

The following user guide outlines the steps that will be done in order to deploy ovn-domain-service.

Control-Plane Deployment
----------------------------

**Prerequisites**

For inventory file variables setup refer to `link <https://github.com/NVIDIA/ovn-isolation-deployment/blob/master/playbooks/control-plane/inventory>`__


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

For inventory file variables setup refer to `link <https://github.com/NVIDIA/ovn-isolation-deployment/blob/master/playbooks/dpu/inventory>`__


**Deployment**

As part of this deployment, Virtual Functions (VFs) will be created to enable SR-IOV (Single Root I/O Virtualization) capabilities. The number of VFs to be created is specified by the `num_vfs` variable defined in the inventory file.


DPU node deployment
----------------------

**Prerequisites**

For inventory file variables setup refer to `link <https://github.com/NVIDIA/ovn-isolation-deployment/blob/master/playbooks/dpu/inventory>`__


**Deployment**

This deployment will configure Open vSwitch (OVS) to be based on DOCA using DPDK (Data Plane Development Kit) and enabling hardware offload capabilities. DOCA version must be at least 8 to support this solution. Ensure that the FLEX_PARSER_ENABLE parameter is set to 8 to enable flexible parser functionality. Additionally, it will:

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


Troubleshooting
----------------------

**DPU Deployment Issues**

- Check Firmware `FLEX_PARSER_ENABLE` - Failed

    If FLEX_PARSER_ENABLE is not set to 8, the check will fail. The flag can be set manually using the command: 'mlxconfig -d /dev/mst/mt41686_pciconf0 set FLEX_PARSER_PROFILE_ENABLE=8'. After resetting the flag the DPU must be rebooted so that the FW change takes effect.

- Error while fetching server API version: Not supported URL scheme http+docker
    
    Requests (python package) has a bug with the latest versions (later than 2.32.0). See here for more details: https://github.com/docker/docker-py/issues/3256
    To solve the problem downgrade the requests package by running this command on the dpu: 'pip install requests==2.31.0'

**Connectivity Issues**

Here are a few things that should be checked when experiencing commectivity issues.

- Supported DOCA version

    For this sulotion we will need DOCA version 8 or higher, ensure supported version using the command 'ovs-vsctl list Open_vSwitch . | grep doca_version'
    If DOCA version is unsuported, reinstall the BF with a new BFB. The command from the host machine is:
	'bfb-install -r <rshim number> -b <bfb image> -c <config options if any>'

- vf and vf representors mismatch

    Check which pci device used by vf and matching vf_rep by using the command 'ip -d link show eth4/pf0vf0'
	If there is a mismatch, see which interfaces are physical using 'mst status -v' 
    Adjust inventory so that physical interfaces are not used.

- OVS interface external id configuration

    We need to match the InterfaceConfig endpoint name with the external_ids of the OVS interface that uses this endpoint. 
    Therefore, we should ensure that the OVS interface's external ID is the same as the InterfaceConfig name. 
    This can be done by setting the interface's external IDs with the following command: 
    'ovs-vsctl set Interface <interface-name> external_ids:iface-id=bf1pf0vf0'

- Detect host/DPU uplink interface 

    Check which are the psychical interface and choose the lower of the two:
	'mst status -v'
