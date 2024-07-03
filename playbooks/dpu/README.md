# OVN-Domains-Deployment


## Pre-requistes

Before proceeding, ensure that you have access to the remote host where Ansible will execute its modules. This can be accomplished by copying your public key to the authorized keys on the remote host using the following command:
`ssh-copy-id -i ~/.ssh/id_rsa.pub remote_user@remote_host`

Verify connectivity with
```sh
ansible all -m ping
```

Install the ovs ansible package
```bash
ansible-galaxy collection install openvswitch.openvswitch
```

## DPU based setup
Refer to the included inventory file for a sample inventory, where you can configure the necessary variables.
Under the `[compute]` section, you need to specify the hosts where the compute nodes will be deployed.
- vfs: VF names should be listed separated by commas,
For example: pf0vf0,pf0vf1
- mac_addresses: mac addresses to be assigned to the VFs separated by commas,
For example: "40:44:00:00:00:01","40:44:00:00:00:02"

Note: The order of "vfs" and "mac_addresses" is significant. Each element's position in the first list corresponds to the name at the same position in the second list.

In the `[compute:vars]` section, assign the following variables:
- num_vfs: number of vfs on compute node
- uplink: uplink interface

In the `[dpu]` section, you need to specify the dpu system where the ovn chassis will be deployed. assign the following variables:
- encap_ip: sets the IP address to be used as the source IP for tunneling (encapsulation)
- ovn_hostname: assign a host name for the ovs instance
- vf_representors: VF representor names should be listed separated by commas. For example: pf0vf0_r, pf0vf1_r
- external_interface_ids: external interface name from ovn separated by commas, for example: bf1pf0vf0,bf1pf0vf1. Note: it is the same name that was provided to the InterfaceConfig API call.

Note: The order of "vf_representors" and "external_interface_ids" is significant. Each element's position in the first list corresponds to the name at the same position in the second list.

In the `[dpu:vars]` section, assign the following variables:
- uplink_name: ovs port uplink name
- bridge_name: ovs phaysical bridge name
- ovs_timeout: ovs add port timeout
- ovs_datapath_type: bridges datapath types (default: netdev)
- mtu: uplink mtu
- hugepages_count: number of 2 MB huge pages
- ovn_sb_db_ip: ovn south bound database ip ( control plane host ip ) that will be used to connect ovn controller with ovn south bound
- ovn_bridge_mappings: maps a physical network name (physnet) to a specific OVS bridge

Additionally, you need to build and specify the ovn_central_image for the OVN controller component. An example of such an image can be found in the [Dockerfile.ovn-controller](TODO add link) file. 

Run the ansible playbook as follows

```bash
ansible-playbook deploy.yaml
```
